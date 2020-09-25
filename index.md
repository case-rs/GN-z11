## logging stdout to journal in Rust

_Notice:_ this example is Rust specific, the method can be applied to any other programming language.  

Systemd simplifies logging with journald. A program can simply write to stdout and it will end up in the journal. 
The ```systemd``` [crate](https://crates.io/crates/systemd) offers a journal function to log messages with priorities yet for maintainance reasons I don't want to use unnecessary crates.

The priorities supported by journal respond 1:1 to the levels used in [syslog](https://linux.die.net/man/3/syslog), as specified in [RFC 5424](https://tools.ietf.org/html/rfc5424#section-6.2.1). 

| Numerical Code | Severity |
|:--------------:|:--------:|
|0| Emergency: system is unusable|
|1| Alert: action must be taken immediately|
|2| Critical: critical conditions|
|3| Error: error conditions|
|4| Warning: warning conditions|
|5| Notice: normal but significant condition|
|6| Informational: informational messages|
|7| Debug: debug-level messages|

              
How are these priorities defined? That's hidden in the [sd-daemon documentation](https://www.freedesktop.org/software/systemd/man/sd-daemon.html).

>#define SD_EMERG   "<0>"  /* system is unusable */
>#define SD_ALERT   "<1>"  /* action must be taken immediately */
>#define SD_CRIT    "<2>"  /* critical conditions */
>#define SD_ERR     "<3>"  /* error conditions */
>#define SD_WARNING "<4>"  /* warning conditions */
>#define SD_NOTICE  "<5>"  /* normal but significant condition */
>#define SD_INFO    "<6>"  /* informational */
>#define SD_DEBUG   "<7>"  /* debug-level messages */

Armed with that information, it's possible to give priorities to certain log messages by simply adding the matching string to the front of the message. For example if one wants to log info messages it's simply done with: 
```rust
use std::io::{stdout, Write};

fn log_info(message: String) -> io::Result<()> {
    SD_INFO = String::from("<6>");
    SD_INFO.push_str(message);
    stdout().write_all(SD_INFO.as_ref());

    Ok(())
}
```

Of course, it's also possibel to create a general function that takes the priority level as a parameter.

