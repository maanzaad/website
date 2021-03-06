---
title: Generic Sharing
redirect_from:
  - /Mono:Runtime:Documentation:GenericSharing/
---

Porting
=======

This describes how to port the generic code sharing infrastructure in Mono. This is in addition to the [Mono Runtime Porting]({{ site.github.url }}/docs/advanced/runtime/docs/mini-porting/) instructions.

Generic class init trampoline
-----------------------------

The generic class init trampoline is very similar to the class init trampoline, with the exception that the VTable of the class to be inited is passed in a register, `MONO_ARCH_VTABLE_REG`. That register can be the same as the IMT register.

The call to the generic class init trampoline is never patched because the VTable is not constant - it depends on the type arguments of the class/method doing the call to the trampoline. For that reason, the trampoline needs a fast path for the case that the class is already inited, i.e. the trampoline needs to check the initialized bit of the MonoVTable and immediately return if it is set. See tramp-x86.c for how to portably figure out the number of the bit in the bit-field that needs to be checked. Since the `MONO_ARCH_VTABLE_REG` is needed by the generic class init trampoline function (in mini-trampolines.c) it needs to be saved by the generic trampoline code in the register save block which is passed to the trampoline. This also applies to the RGCTX fetch trampoline (see below).

Like the normal class init trampoline, the result of the generic class init trampoline function is not a pointer to code that needs to be jumped to. Instead, just like for the class init trampoline, the generic trampoline code must return to the caller instead of jumping to the returned value. The same applies for the RGCTX fetch trampoline (see below).

RGCTX register
--------------

Generic shared code needs access to type information. This information is contained in a RGCTX for non-generic methods and in an MRGCTX for generic methods. It is passed in one of several ways, depending on the type of the called method:

1.  Non-generic non-static methods of reference types have access to the RGCTX via the "this" argument (this-\>vtable-\>rgctx).

1.  Non-generic static methods of reference types and non-generic methods of value types need to be passed a pointer to the caller's class's VTable in the MONO\_ARCH\_RGCTX\_REG register.

1.  Generic methods need to be passed a pointer to the MRGCTX in the `MONO_ARCH_RGCTX_REG` register.

The `MONO_ARCH_RGCTX_REG` must not be clobbered by trampolines.

`MONO_ARCH_RGCTX_REG` can be the same as the IMT register for now, but this might change in the future when we implement virtual generic method calls (more) efficiently.

This register lifetime starts at the call site that loads it and ends in the callee prologue when it is either discarded or stored into a local variable.

It's better to avoid registers used for argument passing for the RGCTX as it would make the code dealing with calling conventions code a lot harder.

Method prologue
---------------

Generic shared code that have a `RGCTX` receive it in `RGCTX_REG`. There must be a check in mono\_arch\_emit\_prolog for MonoCompile::rgctx\_var and if set store it. See mini-x86.c for reference.

Dealing with types
------------------

Types passed to arch functions might be type parameters (`MONO_TYPE_(M)VAR`) if the `MonoGenericSharingContext*` argument is non-NULL. For example, an argument or return type in a method passed to `mono_arch_find_this_argument()` could be a `MONO_TYPE_VAR`. To guard for that case use `mini_get_basic_type_from_generic()` on the type. See `get_call_info()` in mini-x86.c, for example.

(M)RGCTX lazy fetch trampoline
------------------------------

The purpose of the lazy fetch trampoline is to fetch a slot from an (M)RGCTX which might not be inited, yet. In the latter case, it needs to go make a transition to unmanaged code to fill the slot. This is the layout of a RGCTX:

         +---------------------------------+
         | next | slot 0 | slot 1 | slot 2 |
         +--|------------------------------+
            |
      +-----+
      |  +---------------------------------
      +->| next | slot 3 | slot 4 | slot 5 ....
         +--|------------------------------
            |
      +-----+
      |  +------------------------------------
      +->| next | slot 10 | slot 11 | slot 12 ....
         +--|---------------------------------
            .
            .
            .

For fetching a slot from a RGCTX the trampoline is passed a pointer (as a normal integer argument) to the VTable. From there it has to fetch the pointer to the RGCTX, which might be null. Then it has to traverse the correct number of "next" links, each of which might be NULL. Arriving at the right array it needs to fetch the slot, which might also be NULL. If any of the NULL cases, the trampoline must transition to unmanaged code to potentially setup the RGCTX and fill the slot. Here is pseudo-code for fetching slot 11:

        ; vtable ptr in r1
        ; fetch RGCTX array 0
        r2 = *(r1 + offsetof(MonoVTable, runtime_generic_context))
        if r2 == NULL goto unmanaged
        ; fetch RGCTX array 1
        r2 = *r2
        if r2 == NULL goto unmanaged
        ; fetch RGCTX array 2
        r2 = *r2
        if r2 == NULL goto unmanaged
        ; fetch slot 11
        r2 = *(r2 + 2 * sizeof (gpointer))
        if r2 == NULL goto unmanaged
        return r2
      unmanaged:
        jump unmanaged_fetch_code

The number of slots in the arrays must be obtained from the function `mono_class_rgctx_get_array_size()`.

The MRGCTX case is different in two aspects. First, the trampoline is not passed a pointer to a VTable, but a pointer directly to the MRGCTX, which is guaranteed not to be NULL (any of the next pointers and any of the slots can be NULL, though). Second, the layout of the first array is slightly different, in that the first two slots are occupied by a pointers to the class's VTable and to the method's method\_inst. The next pointer is in the third slot and the first actual slot, "slot 0", in the fourth:

         +--------------------------------------------------------+
         | vtable | method_inst | next | slot 0 | slot 1 | slot 2 |
         +-------------------------|------------------------------+
                                   .
                                   .

All other arrays have the same layout as the RGCTX ones, except possibly for their length.

The function to create the trampoline, mono\_arch\_create\_rgctx\_lazy\_fetch\_trampoline(), gets passed an encoded slot number. Use the macro `MONO_RGCTX_SLOT_IS_MRGCTX` to query whether a trampoline for an MRGCTX is needed, as opposed to one for a RGCTX. Use `MONO_RGCTX_SLOT_INDEX` to get the index of the slot (like 2 for "slot 2" as above). The unmanaged fetch code is yet another trampoline created via `mono_arch_create_specific_trampoline()`, of type `MONO_TRAMPOLINE_RGCTX_LAZY_FETCH`. It's given the slot number as the trampoline argument. In addition, the pointer to the VTable/MRGCTX is passed in `MONO_ARCH_VTABLE_REG` (like the VTable to the generic class init trampoline - see above).

Like the class init and generic class init trampolines, the RGCTX fetch trampoline code doesn't return code that must be jumped to, so, like for those trampolines (see above), the generic trampoline code must do a normal return instead.

Getting generics information about a stack frame
================================================

If a method is compiled with generic sharing, its `MonoJitInfo` has `has_generic_jit_info` set. In that case, the `MonoJitInfo` is followed (after the `MonoJitExceptionInfo` array) by a `MonoGenericJitInfo`.

The MonoGenericJitInfo contains information about the location of the this/vtable/MRGCTX variable, if the has\_this flag is set. If that is the case, there are two possibilities:

1.  this\_in\_reg is set. this\_reg is the number of the register where the variable is stored.

1.  this\_in\_reg is not set. The variable is stored at offset this\_offset from the address in the register with number this\_reg.

The variable can either point to the "this" object, to a vtable or to an MRGCTX:

1.  If the method is a non-generic non-static method of a reference type, the variable points to the "this" object.

1.  If the method is a non-generic static method or a non-generic method of a value type, the variable points to the vtable of the class.

1.  If the method is a generic method, the variable points to the MRGCTX of the method.

Layout of the MRGCTX
====================

The MRGCTX is a structure that starts with `MonoMethodRuntimeGenericContext`, which contains a pointer to the vtable of the class and a pointer to the `MonoGenericInst` with the type arguments for the method.

Blog posts about generic code sharing
=====================================

-   [September 2007: Generics Sharing in Mono](http://schani.wordpress.com/2007/09/22/generics-sharing-in-mono/)
-   [October 2007: The Trouble with Shared Generics](http://schani.wordpress.com/2007/10/12/the-trouble-with-shared-generics/)
-   [October 2007: A Quick Generics Sharing Update](http://schani.wordpress.com/2007/10/15/a-quick-generics-sharing-update/)
-   [January 2008: Other Types](http://schani.wordpress.com/2008/01/29/other-types/)
-   [February 2008: Generic Types Are Lazy](http://schani.wordpress.com/2008/02/25/generic-types-are-lazy/)
-   [March 2008: Sharing Static Methods](http://schani.wordpress.com/2008/03/10/sharing-static-methods/)
-   [April 2008: Sharing Everything And Saving Memory](http://schani.wordpress.com/2008/04/22/sharing-everything-and-saving-memory/)
-   [June 2008: Sharing Generic Methods](http://schani.wordpress.com/2008/06/02/sharing-generic-methods/)
-   [June 2008: Another Generic Sharing Update](http://schani.wordpress.com/2008/06/27/another-generic-sharing-update/)


