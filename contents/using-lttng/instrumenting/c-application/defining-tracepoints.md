---
id: defining-tracepoints
---

As written in [Tracepoint provider](#doc-tracepoint-provider),
tracepoints are defined using the
`TRACEPOINT_EVENT()` macro. Each tracepoint, when called using the
`tracepoint()` macro in the actual application's source code, generates
a specific event type with its own fields.

Let's have another look at the example above, with a few added comments:

~~~ c
TRACEPOINT_EVENT(
    /* tracepoint provider name */
    my_provider,

    /* tracepoint/event name */
    my_first_tracepoint,

    /* list of tracepoint arguments */
    TP_ARGS(
        int, my_integer_arg,
        char*, my_string_arg
    ),

    /* list of fields of eventual event  */
    TP_FIELDS(
        ctf_string(my_string_field, my_string_arg)
        ctf_integer(int, my_integer_field, my_integer_arg)
    )
)
~~~

The tracepoint provider name must match the name of the tracepoint
provider in which this tracepoint is defined
(see [Tracepoint provider](#doc-tracepoint-provider)). In other words,
always use the same string as the value of `TRACEPOINT_PROVIDER` above.

The tracepoint name will become the event name once events are recorded
by the LTTng-UST tracer. It must follow the tracepoint provider name
syntax: start with a letter and contain either letters, numbers or
underscores. Two tracepoints under the same provider cannot have the
same name, i.e. you cannot overload a tracepoint like you would
overload functions and methods in C++/Java.

<div class="tip">
<p><span class="t">Note:</span>The concatenation of the tracepoint
provider name and the tracepoint name cannot exceed 254 characters. If
it does, the instrumented application will compile and run, but LTTng
will issue multiple warnings and you could experience serious problems.</p>
</div>

The list of tracepoint arguments gives this tracepoint its signature:
see it like the declaration of a C function. The format of `TP_ARGS()`
arguments is: C type, then argument name; repeat as needed, up to ten
times. For example, if we were to replicate the signature of C standard
library's `fseek()`, the `TP_ARGS()` part would look like:

~~~ c
    TP_ARGS(
        FILE*, stream,
        long int, offset,
        int, origin
    ),
~~~

Of course, you will need to include appropriate header files before
the `TRACEPOINT_EVENT()` macro calls if any argument has a complex type.

`TP_ARGS()` may not be omitted, but may be empty. `TP_ARGS(void)` is
also accepted.

The list of fields is where the fun really begins. The fields defined
in this list will be the fields of the events generated by the execution
of this tracepoint. Each tracepoint field definition has a C
_argument expression_ which will be evaluated when the execution reaches
the tracepoint. Tracepoint arguments _may be_ used freely in those
argument expressions, but they _don't_ have to.

There are several types of tracepoint fields available. The macros to
define them are given and explained in the
[LTTng-UST library reference](#doc-liblttng-ust-tp-fields) section.

Field names must follow the standard C identifier syntax: letter, then
optional sequence of letters, numbers or underscores. Each field must have
a different name.

Those `ctf_*()` macros are added to the `TP_FIELDS()` part of
`TRACEPOINT_EVENT()`. Note that they are not delimited by commas.
`TP_FIELDS()` may be empty, but the `TP_FIELDS(void)` form is _not_
accepted.

The following snippet shows how argument expressions may be used in
tracepoint fields and how they may refer freely to tracepoint arguments.

~~~ c
/* for struct stat */
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

TRACEPOINT_EVENT(
    my_provider,
    my_tracepoint,
    TP_ARGS(
        int, my_int_arg,
        char*, my_str_arg,
        struct stat*, st
    ),
    TP_FIELDS(
        /* simple integer field with constant value */
        ctf_integer(
            int,                    /* field C type */
            my_constant_field,      /* field name */
            23 + 17                 /* argument expression */
        )

        /* my_int_arg tracepoint argument */
        ctf_integer(
            int,
            my_int_arg_field,
            my_int_arg
        )

        /* my_int_arg squared */
        ctf_integer(
            int,
            my_int_arg_field2,
            my_int_arg * my_int_arg
        )

        /* sum of first 4 characters of my_str_arg */
        ctf_integer(
            int,
            sum4,
            my_str_arg[0] + my_str_arg[1] +
            my_str_arg[2] + my_str_arg[3]
        )

        /* my_str_arg as string field */
        ctf_string(
            my_str_arg_field,       /* field name */
            my_str_arg              /* argument expression */
        )

        /* st_size member of st tracepoint argument, hexadecimal */
        ctf_integer_hex(
            off_t,                  /* field C type */
            size_field,             /* field name */
            st->st_size             /* argument expression */
        )

        /* st_size member of st tracepoint argument, as double */
        ctf_float(
            double,                 /* field C type */
            size_dbl_field,         /* field name */
            (double) st->st_size    /* argument expression */
        )

        /* half of my_str_arg string as text sequence */
        ctf_sequence_text(
            char,                   /* element C type */
            half_my_str_arg_field,  /* field name */
            my_str_arg,             /* argument expression */
            size_t,                 /* length expression C type */
            strlen(my_str_arg) / 2  /* length expression */
        )
    )
)
~~~

As you can see, having a custom argument expression for each field
makes tracepoints very flexible for tracing a user space C application.
This tracepoint definition is reused later in this guide, when
actually using tracepoints in a user space application.