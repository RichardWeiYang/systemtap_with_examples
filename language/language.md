systemtap is so powerful, because we could write scripts to program the
behavior according to our purpose.

> The magic all exists in the script.

The script is like c/awk. The general idea is simple, while there are still
different concepts need to understand before we can handle the script.

Based on my humble understanding, a programming language could be divided into
several parts:

* [Syntax](language/syntax.md)
* [Variables](language/variables.md)
* [Control Flow](language/control_flow.md)
* [Function & Macro](language/function_macro.md)

Besides these, systemtap has several special parts:

* [TypeCase](language/typecast.md)
* [Guru Mode](language/guru_mode.md)
* [Probe Point](language/probe_point.md)
* [Predefined Functions](language/predefined_functions.md)

When I almost finish this serials, I found a very good [Language Reference][1]
from RH. I believe this one is more sophisticated than my serials. I would
recommend you to read that and my serials would refer to that one in some
points.

BTW, I am glad to see the author was my colleague. Thanks to IBM.

[1]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/systemtap_language_reference/
