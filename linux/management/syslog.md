# syslog

## Basic Introduction
syslog is one mechanism, any upper application can make use of this to log something.

## Basic Usage
- void openlog(const char *ident, int option, int facility);
- void syslog(int priority, const char *format, ...);
- void closelog(void);

## Basic Example
```c
#include <syslog.h>

int main(int argc, char **argv)
{
    openlog("syslog_test", LOG_PID, LOG_DAEMON);

    syslog(LOG_EMERG, "system is unusable");
    syslog(LOG_ALERT, "action must be taken immediately");
    syslog(LOG_CRIT, "critical conditions");
    syslog(LOG_ERR, "error conditions");
    syslog(LOG_WARNING, "warning conditions");
    syslog(LOG_NOTICE, "normal, but significant, condition");
    syslog(LOG_INFO, "informational message");
    syslog(LOG_DEBUG, "debug-level message");

    closelog();

    return 0;
}

```

## Output

/var/log/messages

- Aug 29 18:10:06 sfu03 syslog_test[24343]: system is unusable
- Aug 29 18:10:06 sfu03 syslog_test[24343]: action must be taken immediately
- Aug 29 18:10:06 sfu03 syslog_test[24343]: critical conditions
- Aug 29 18:10:06 sfu03 syslog_test[24343]: error conditions
- Aug 29 18:10:06 sfu03 syslog_test[24343]: warning conditions
- Aug 29 18:10:06 sfu03 syslog_test[24343]: normal, but significant, condition
- Aug 29 18:10:06 sfu03 syslog_test[24343]: informational message

## the difference between syslog/rsyslog/syslog-ng

The Syslog project was the very first project. It started in 1980. It is the root project to Syslog protocol. 
At this time Syslog is a very simple protocol. At the beginning it only supports UDP for transport, so that it does not guarantee the delivery of the messages.

Next came syslog-ng in 1998. It extends basic syslog protocol with new features like:
- content-based filtering
- Logging directly into a database
- TCP for transport
- TLS encryption

Next came Rsyslog in 2004. It extends syslog protocol with new features like:

- RELP Protocol support
- Buffered operation support
