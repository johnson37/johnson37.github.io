# syslogd

```c
int syslogd_main(int argc UNUSED_PARAM, char **argv)
{
    write_pidfile("/var/run/syslogd.pid");
    do_syslogd();
}

static void do_syslogd(void)
{
    sock_fd = create_socket();
	while(1){
 read_again:
        sz = read(sock_fd, recvbuf, MAX_READ - 1);
        while (1) {
            if (sz == 0)
                goto read_again;
            /* man 3 syslog says: "A trailing newline is added when needed".
             * However, neither glibc nor uclibc do this:
             * syslog(prio, "test")   sends "test\0" to /dev/log,
             * syslog(prio, "test\n") sends "test\n\0".
             * IOW: newline is passed verbatim!
             * I take it to mean that it's syslogd's job
             * to make those look identical in the log files. */
            if (recvbuf[sz-1] != '\0' && recvbuf[sz-1] != '\n')
                break;
            sz--;
        }
        if (!ENABLE_FEATURE_REMOTE_LOG || (option_mask32 & OPT_locallog)) {
            recvbuf[sz] = '\0'; /* ensure it *is* NUL terminated */
            split_escape_and_log(recvbuf, sz);
        }
	}
}

static NOINLINE int create_socket(void)
{
    struct sockaddr_un sunx;
    int sock_fd;
    char *dev_log_name;

    memset(&sunx, 0, sizeof(sunx));
    sunx.sun_family = AF_UNIX;

    /* Unlink old /dev/log or object it points to. */
    /* (if it exists, bind will fail) */
    strcpy(sunx.sun_path, "/dev/log");
    dev_log_name = xmalloc_follow_symlinks("/dev/log");
    if (dev_log_name) {
        safe_strncpy(sunx.sun_path, dev_log_name, sizeof(sunx.sun_path));
        free(dev_log_name);
    }
    unlink(sunx.sun_path);

    sock_fd = xsocket(AF_UNIX, SOCK_DGRAM, 0);
    xbind(sock_fd, (struct sockaddr *) &sunx, sizeof(sunx));
    chmod("/dev/log", 0666);

    return sock_fd;
}

static void split_escape_and_log(char *tmpbuf, int len)
{
    tmpbuf += len;
    while (p < tmpbuf) {
        /* Now log it */
        if (LOG_PRI(pri) < G.logLevel)
            timestamp_and_log(pri, G.parsebuf, q - G.parsebuf);
	}
}

static void timestamp_and_log(int pri, char *msg, int len)
{

    if (len < 16 || msg[3] != ' ' || msg[6] != ' '
     || msg[9] != ':' || msg[12] != ':' || msg[15] != ' '
    ) {
        time(&now);
        timestamp = ctime(&now) + 4; /* skip day of week */
    } else {
        now = 0;
        timestamp = msg;
        msg += 16;
    }
    timestamp[15] = '\0';

    /* Log message locally (to file or shared mem) */
    log_locally(now, G.printbuf);

}

static void log_locally(time_t now, char *msg)
{
	//open file
	//write
}
```