---
layout: post
title: Linux select
date: 2014-04-30 21:02:08.000000000 +08:00
type: post
published: true
status: publish
categories:
- Linux
tags:
- select
meta:
  _edit_last: '1'
  _wp_old_slug: linux%e4%b8%adselect-io%e5%a4%8d%e7%94%a8%e6%9c%ba%e5%88%b6
author:
  login: oxnz
  email: yunxinyi@gmail.com
  display_name: Will Z
  first_name: Will
  last_name: Z
---
<h2 id="intro">介绍</h2>
<p>最近在看 Linux/Unix 网络编程，谢了三篇关于 select、poll 和 epoll 的文章。本篇介绍select，<a title="Linux poll" href="http://xinyi.sourceforge.net/linux-poll/" target="_blank">第二篇</a>介绍poll，<a title="Linux epoll" href="http://xinyi.sourceforge.net/linux-epool/" target="_blank">第三篇</a>介绍 epoll。</p>
<ol>
<li><a href="#intro">介绍</a>
<ol>
<li><a href="#select-pic">select 原理图</a></li>
<li><a href="#man-select">man select</a></li>
</ol>
</li>
<li><a href="#example-program">实例程序</a>
<ol>
<li><a href="#input-detect">输入监测</a></li>
<li><a href="#web-server">web 服务器</a></li>
</ol>
</li>
<li><a href="summary">总结</a></li>
<li><a href="#refs">参考</a></li>
</ol>
<h2 id="select-pic">select 原理图</h2>
<p>[caption id="attachment_1395" align="aligncenter" width="475"]<a href="https://blog-oxnz.rhcloud.com/wp-content/uploads/2014/04/2429699_1331492431cuPx.gif"><img class="wp-image-1395 size-full" src="{{ site.baseurl }}/assets/2429699_1331492431cuPx.gif" alt="select 原理图" width="475" height="467" /></a> select 原理图，摘自<a href="http://publib.boulder.ibm.com/infocenter/iseries/v5r3/index.jsp?topic=%2Frzab6%2Frzab6xnonblock.htm">IBM iSeries 信息中心</a>[/caption]</p>
<p><!--more--></p>
<h2 id="man-select">man 手册</h2>
<blockquote>
<h2>Name</h2>
<p>select, pselect, FD_CLR, FD_ISSET, FD_SET, FD_ZERO - synchronous I/O multiplexing</p>
<h2>Synopsis</h2>
<pre class="lang:default decode:true ">/* According to POSIX.1-2001 */
#include &lt;sys/select.h&gt;

/* According to earlier standards */
#include &lt;sys/time.h&gt;
#include &lt;sys/types.h&gt;
#include &lt;unistd.h&gt;

int select(int nfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval *timeout);

void FD_CLR(int fd, fd_set *set);
int  FD_ISSET(int fd, fd_set *set);
void FD_SET(int fd, fd_set *set);
void FD_ZERO(fd_set *set);

#include &lt;sys/select.h&gt;

int pselect(int nfds, fd_set *readfds, fd_set *writefds,
            fd_set *exceptfds, const struct timespec *timeout,
            const sigset_t *sigmask);</pre>
<p>Feature Test Macro Requirements for glibc (see <b><a href="http://linux.die.net/man/7/feature_test_macros">feature_test_macros</a></b>(7)):</p>
<dl compact="compact">
<dt><b>pselect</b>(): _POSIX_C_SOURCE &gt;= 200112L || _XOPEN_SOURCE &gt;= 600</dt>
</dl>
<h2>Description</h2>
<p><b>select</b>() 和 <b>pselect</b>() 运行程序来监控多个文件描述符，等待至少一个文件描述符对一些类型的 I/O 操作 (例如输入)变为就绪状态。文件描述符就绪意味着可以进行对应的操作 (e.g., <b><a style="color: #660000;" href="http://linux.die.net/man/2/read">read</a></b>(2))而不阻塞。</p>
<p><b>select</b>() 和 <b>pselect</b>() 的操作是相同的, 除了以下三点不同:</p>
<dl compact="compact">
<dt>(i)</dt>
<dd><b>select</b>() 使用一个 <i>struct timeval</i>  结构体的超时值(包含秒和微秒), 而 <b>pselect</b>() 使用一个 <i>struct timespec</i> 结构体做超时值 (包含秒和纳秒).</dd>
<dt>(ii)</dt>
<dd><b>select</b>() 可能会更新 <i>timeout</i> 参数来反映剩余的超时时间。<b>pselect</b>() 不改变此参数。</dd>
<dt>(iii)</dt>
<dd><b>select</b>() 没有 <i>sigmask</i> 参数, 行为和使用 NULL <i>sigmask</i> 调用 <b>pselect</b>() 一样。</dd>
</dl>
<p>单个独立的文件描述符集合被监控。<i>readfds</i> 中的被监控是否有字符可以读取(更精确的说是读操作是不会阻塞；特别的，到达文件尾的文件描述符也会变为就绪状态),<i>writefds</i> 被监控写不会阻塞，<i>exceptfds</i> 被监控是否出现异常。退出时，这些集合被修改以指示具体哪些文件描述符有实际的状态更改。这三个集合每个都可以为 NULL 表示不需要监控特定类型的事件。</p>
<p>四个宏被用来操作这些集合。<b>FD_ZERO</b>() 清空集合。<b>FD_SET</b>() 和 <b>FD_CLR</b>() 分别从集合中添加和移除给定的问及描述符。<b>FD_ISSET</b>() 测试一个文件描述符是否属于某个集合；这在<b>select</b>() 返回的时候非常有用。</p>
<p><i>nfds</i> 是三个集合中最大的文件描述符+1。</p>
<p><i>timeout</i> 参数指定了 <b>select</b>() 应当阻塞等待文件描述符变为就绪状态的最小间隔。(This interval will be rounded up to the system clock granularity, and kernel scheduling delays mean that the blocking interval may overrun by a small amount.) 如果 <i>timeval</i> 结构体的两个成员都为0，则 <b>select</b>() 立即返回。(This is useful for polling.) 如果 <i>timeout</i> 为 NULL (no timeout), <b>select</b>() 将会无限期阻塞。</p>
<p><i>sigmask</i> 是一个指向 signal mask (see <b><a style="color: #660000;" href="http://linux.die.net/man/2/sigprocmask">sigprocmask</a></b>(2)) 的指针; 如果它不为 NULL, 则 <b>pselect</b>() 首先使用 <i>sigmask</i> 指向的 signal mask 替换当前的 signal mask, 然后执行 "select" 函数, 最后回复原始的 signal mask.除了 <i>timeout</i> 参数的精度不同之外，下面的 <b>pselect</b>() 调用:</p>
<dl compact="compact">
<dt>
</dt>
</dl>
<pre class="code">ready = pselect(nfds, &amp;readfds, &amp;writefds, &amp;exceptfds,
                timeout, &amp;sigmask);</pre>
<p>等同于原子执行以下调用:</p>
<pre class="code">sigset_t origmask;

pthread_sigmask(SIG_SETMASK, &amp;sigmask, &amp;origmask);
ready = select(nfds, &amp;readfds, &amp;writefds, &amp;exceptfds, timeout);
pthread_sigmask(SIG_SETMASK, &amp;origmask, NULL);</pre>
<p><b>pselect</b>() 存在的原因在于如果有人想要等待一个文件描述符就绪或者一个信号发生，就需要一个原子测试来防止竞争条件(to prevent race conditions)。(Suppose the signal handler sets a global flag and returns. Then a test of this global flag followed by a call of <b>select</b>() could hang indefinitely if the signal arrived just after the test but just before the call. By contrast, <b>pselect</b>() allows one to first block signals, handle the signals that have come in, then call <b>pselect</b>() with the desired <i>sigmask</i>, avoiding the race.)</p>
<p><b>The timeout</b></p>
<dl compact="compact">
<dt>The time structures involved are defined in <i>&lt;<a href="http://linux.die.net/include/sys/time.h" rel="nofollow">sys/time.h</a>&gt;</i> and look like</dt>
<dd>
<pre class="code">struct timeval {
    long    tv_sec;         /* seconds */
    long    tv_usec;        /* microseconds */
};</pre>
</dd>
<dt>and</dt>
<dd>
<pre class="code">struct timespec {
    long    tv_sec;         /* seconds */
    long    tv_nsec;        /* nanoseconds */
};</pre>
</dd>
<dt>(However, see below on the POSIX.1-2001 versions.)有些代码使用三个空集合、nfds为0，<em>timeout</em>不为NULL<i> 来调用</i> <b>select</b>()作为一种可移植的方法来达到亚秒精度的睡眠(subsecond precision)。在 Linux 系统中, <b>select</b>() 修改 <i>timeout</i> 来反映剩余的睡眠时间; 大多数其他实现并无此操作。(POSIX.1-2001 permits either behavior.) This causes problems both when Linux code which reads <i>timeout</i> is ported to other operating systems, and when code is ported to Linux that reuses a <i>struct timeval</i> for multiple <b>select</b>()s in a loop without reinitializing it. <span style="color: #ff0000;">最好还是把调用 <b>select</b>() 之后的 <i>timeout</i> 当做未定义的值。</span></dt>
</dl>
<h2>Return Value</h2>
<p>成功时, <b>select</b>() 和 <b>pselect</b>() 返回三个集合中文件描述符数目(也就是, 三个集合 <i>readfds</i>, <i>writefds</i>, <i>exceptfds </i>中被设置的位的总数) ；超时的时候返回0。 有错误的情况下返回 -1， 并且 <i>errno</i> 被适当设置；集合和 <i>timeout</i> 变成未定义的，所以<span style="color: #ff0000;">不要在错误发生之后依赖它们的内容</span>。</p>
<h2>Errors</h2>
<dl compact="compact">
<dt><b>EBADF</b></dt>
<dd>集合中含有无效的文件描述符。(可能是有的文件描述符已经被关闭，或者有错误发生。)</dd>
<dt><b>EINTR</b></dt>
<dd>A signal was caught; see <b><a style="color: #660000;" href="http://linux.die.net/man/7/signal">signal</a></b>(7).</dd>
<dt><b>EINVAL</b></dt>
<dd><i>nfds</i> 为负或者 <i>timeout</i> 包含无效值。</dd>
<dt><b>ENOMEM</b></dt>
<dd>为内部表分配内存失败。</dd>
</dl>
<h2>Versions</h2>
<p><b>pselect</b>() was added to Linux in kernel 2.6.16. Prior to this, <b>pselect</b>() was emulated in glibc (but see BUGS).</p>
<h2>Conforming To</h2>
<p><b>select</b>() conforms to POSIX.1-2001 and 4.4BSD (<b>select</b>() first appeared in 4.2BSD). Generally portable to/from non-BSD systems supporting clones of the BSD socket layer (including System V variants). However, note that the System V variant typically sets the timeout variable before exit, but the BSD variant does not.</p>
<p><b>pselect</b>() is defined in POSIX.1g, and in POSIX.1-2001.</p>
<h2>Notes</h2>
<p><i>fd_set</i> 是一个固定大小的缓冲区。调用 <b>FD_CLR</b>() 或者 <b>FD_SET</b>() 的 <i>fd</i> 值为负或者不小于 <b>FD_SETSIZE</b> 将会导致未定义的行为。而且, POSIX 要求 <i>fd</i> 为有效的文件描述符。</p>
<p>Concerning the types involved, the classical situation is that the two fields of a <i>timeval</i> structure are typed as <i>long</i> (as shown above), and the structure is defined in<i>&lt;<a style="color: #660000;" href="http://linux.die.net/include/sys/time.h" rel="nofollow">sys/time.h</a>&gt;</i>. The POSIX.1-2001 situation is</p>
<dl compact="compact">
<dd>
<pre class="code">struct timeval {
    time_t         tv_sec;     /* seconds */
    suseconds_t    tv_usec;    /* microseconds */
};</pre>
</dd>
<dt>where the structure is defined in <i>&lt;<a href="http://linux.die.net/include/sys/select.h" rel="nofollow">sys/select.h</a>&gt;</i> and the data types <i>time_t</i> and <i>suseconds_t</i> are defined in <i>&lt;<a style="color: #660000;" href="http://linux.die.net/include/sys/types.h" rel="nofollow">sys/types.h</a>&gt;</i>.Concerning prototypes, the classical situation is that one should include <i>&lt;<a style="color: #660000;" href="http://linux.die.net/include/time.h" rel="nofollow">time.h</a>&gt;</i> for <b>select</b>(). The POSIX.1-2001 situation is that one should include <i>&lt;<a style="color: #660000;" href="http://linux.die.net/include/sys/select.h" rel="nofollow">sys/select.h</a>&gt;</i> for <b>select</b>() and <b>pselect</b>().Libc4 and libc5 do not have a <i>&lt;<a style="color: #660000;" href="http://linux.die.net/include/sys/select.h" rel="nofollow">sys/select.h</a>&gt;</i> header; under glibc 2.0 and later this header exists. Under glibc 2.0 it unconditionally gives the wrong prototype for <b>pselect</b>(). Under glibc 2.1 to 2.2.1 it gives <b>pselect</b>() when <b>_GNU_SOURCE</b> is defined. Since glibc 2.2.2 the requirements are as shown in the SYNOPSIS.</dt>
</dl>
<p><b>Multithreaded applications</b></p>
<dl compact="compact">
<dt><span style="color: #ff0000;">If a file descriptor being monitored by <b>select</b>() is closed in another thread, the result is unspecified.</span> On some UNIX systems, <b>select</b>() unblocks and returns, with an indication that the file descriptor is ready (a subsequent I/O operation will likely fail with an error, unless another the file descriptor reopened between the time <b>select</b>() returned and the I/O operations was performed). On Linux (and some other systems), closing the file descriptor in another thread has no effect on <b>select</b>().<span style="color: #ff0000;"> 总之，任何依赖于特定的某种情况的程序被认为是有缺陷的(In summary, any application that relies on a particular behavior in this scenario must be considered buggy)。</span></dt>
</dl>
<p><b>Linux notes</b></p>
<dl compact="compact">
<dt>The <b>pselect</b>() interface described in this page is implemented by glibc. The underlying Linux system call is named <b>pselect6</b>(). This system call has somewhat different behavior from the glibc wrapper function.The Linux <b>pselect6</b>() system call modifies its <i>timeout</i> argument. However, the glibc wrapper function hides this behavior by using a local variable for the timeout argument that is passed to the system call. Thus, the glibc <b>pselect</b>() function does not modify its <i>timeout</i> argument; this is the behavior required by POSIX.1-2001.The final argument of the <b>pselect6</b>() system call is not a <i>sigset_t *</i> pointer, but is instead a structure of the form:</dt>
<dd>
<pre class="code">struct {
    const sigset_t *ss;     /* Pointer to signal set */
    size_t          ss_len; /* Size (in bytes) of object pointed
                               to by 'ss' */
};</pre>
</dd>
<dt>This allows the system call to obtain both a pointer to the signal set and its size, while allowing for the fact that most architectures support a maximum of 6 arguments to a system call.</dt>
</dl>
<h2>Bugs</h2>
<p>Glibc 2.0 provided a version of <b>pselect</b>() that did not take a <i>sigmask</i> argument.</p>
<p>Starting with version 2.1, glibc provided an emulation of <b>pselect</b>() that was implemented using <b><a style="color: #660000;" href="http://linux.die.net/man/2/sigprocmask" rel="nofollow">sigprocmask</a></b>(2) and <b>select</b>(). This implementation remained vulnerable to the very race condition that <b>pselect</b>() was designed to prevent. Modern versions of glibc use the (race-free) <b>pselect</b>() system call on kernels where it is provided.</p>
<p>On systems that lack <b>pselect</b>(), reliable (and more portable) signal trapping can be achieved using the self-pipe trick. In this technique, a signal handler writes a byte to a pipe whose other end is monitored by <b>select</b>() in the main program. (To avoid possibly blocking when writing to a pipe that may be full or reading from a pipe that may be empty, nonblocking I/O is used when reading from and writing to the pipe.)</p>
<p>Under Linux, <b>select</b>() may report a socket file descriptor as "ready for reading", while nevertheless a subsequent read blocks. This could for example happen when data has arrived but upon examination has wrong checksum and is discarded. There may be other circumstances in which a file descriptor is spuriously reported as ready. <span style="color: #ff0000;">Thus it may be safer to use <b>O_NONBLOCK</b> on sockets that should not block.</span></p>
<p>On Linux, <b>select</b>() also modifies <i>timeout</i> if the call is interrupted by a signal handler (i.e., the <b>EINTR</b> error return). This is not permitted by POSIX.1-2001. The Linux <b>pselect</b>() system call has the same behavior, but the glibc wrapper hides this behavior by internally copying the <i>timeout</i> to a local variable and passing that variable to the system call.</p>
<h2>Example</h2>
<pre class="code">#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;
#include &lt;sys/time.h&gt;
#include &lt;sys/types.h&gt;
#include &lt;unistd.h&gt;

int
main(void)
{
    fd_set rfds;
    struct timeval tv;
    int retval;

   /* Watch stdin (fd 0) to see when it has input. */
    FD_ZERO(&amp;rfds);
    FD_SET(0, &amp;rfds);

   /* Wait up to five seconds. */
    tv.tv_sec = 5;
    tv.tv_usec = 0;

   retval = select(1, &amp;rfds, NULL, NULL, &amp;tv);
    /* Don't rely on the value of tv now! */

   if (retval == -1)
        perror("select()");
    else if (retval)
        printf("Data is available now.n");
        /* FD_ISSET(0, &amp;rfds) will be true. */
    else
        printf("No data within five seconds.n");

   exit(EXIT_SUCCESS);
}</pre>
<h2>See Also</h2>
<p><b><a style="color: #660000;" href="http://linux.die.net/man/2/accept">accept</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/2/connect">connect</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/2/poll">poll</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/2/read" rel="nofollow">read</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/2/recv">recv</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/2/send">send</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/2/sigprocmask" rel="nofollow">sigprocmask</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/2/write">write</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/7/epoll">epoll</a></b>(7), <b><a style="color: #660000;" href="http://linux.die.net/man/7/time">time</a></b>(7)</p>
<p>For a tutorial with discussion and examples, see <b><a style="color: #660000;" href="http://linux.die.net/man/2/select_tut">select_tut</a></b>(2).</p>
<h2>Referenced By</h2>
<p><b><a style="color: #660000;" href="http://linux.die.net/man/2/accept4" rel="nofollow">accept4</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/2/alarm" rel="nofollow">alarm</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/3/avc_netlink_open" rel="nofollow">avc_netlink_open</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/8/coroipc_overview" rel="nofollow">coroipc_overview</a></b>(8), <b><a style="color: #660000;" href="http://linux.die.net/man/3/curl_easy_recv" rel="nofollow">curl_easy_recv</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/curl_easy_send" rel="nofollow">curl_easy_send</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/8/distcache" rel="nofollow">distcache</a></b>(8), <b><a style="color: #660000;" href="http://linux.die.net/man/3/dnet" rel="nofollow">dnet</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/2/epoll_wait" rel="nofollow">epoll_wait</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/3/ev" rel="nofollow">ev</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/event" rel="nofollow">event</a></b>(3),<b><a style="color: #660000;" href="http://linux.die.net/man/2/eventfd" rel="nofollow">eventfd</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/1/explain" rel="nofollow">explain</a></b>(1), <b><a style="color: #660000;" href="http://linux.die.net/man/3/explain" rel="nofollow">explain</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/explain_select" rel="nofollow">explain_select</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/explain_select_or_die" rel="nofollow">explain_select_or_die</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/2/fcntl" rel="nofollow">fcntl</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/8/haveged" rel="nofollow">haveged</a></b>(8), <b><a style="color: #660000;" href="http://linux.die.net/man/3/ident" rel="nofollow">ident</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/ieee1284_get_irq_fd" rel="nofollow">ieee1284_get_irq_fd</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/imclient" rel="nofollow">imclient</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/7/inotify" rel="nofollow">inotify</a></b>(7),<b><a style="color: #660000;" href="http://linux.die.net/man/3/iopause" rel="nofollow">iopause</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/ipq_read" rel="nofollow">ipq_read</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/1/jstest" rel="nofollow">jstest</a></b>(1), <b><a style="color: #660000;" href="http://linux.die.net/man/5/ldap.conf" rel="nofollow">ldap.conf</a></b>(5), <b><a style="color: #660000;" href="http://linux.die.net/man/3/ldap_result" rel="nofollow">ldap_result</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/ldap_set_option" rel="nofollow">ldap_set_option</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/libnids" rel="nofollow">libnids</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/libxradius" rel="nofollow">libxradius</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/2/migrate_pages" rel="nofollow">migrate_pages</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/7/mq_overview" rel="nofollow">mq_overview</a></b>(7),<b><a style="color: #660000;" href="http://linux.die.net/man/2/nal_connection_new" rel="nofollow">nal_connection_new</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/2/nal_selector_new" rel="nofollow">nal_selector_new</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/7/nfsd" rel="nofollow">nfsd</a></b>(7), <b><a style="color: #660000;" href="http://linux.die.net/man/5/nss_ldap" rel="nofollow">nss_ldap</a></b>(5), <b><a style="color: #660000;" href="http://linux.die.net/man/1/nttcp" rel="nofollow">nttcp</a></b>(1), <b><a style="color: #660000;" href="http://linux.die.net/man/5/pam_ldap" rel="nofollow">pam_ldap</a></b>(5), <b><a style="color: #660000;" href="http://linux.die.net/man/2/pause" rel="nofollow">pause</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/3/pcap_get_selectable_fd" rel="nofollow">pcap_get_selectable_fd</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/2/perf_event_open" rel="nofollow">perf_event_open</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/2/perfmonctl" rel="nofollow">perfmonctl</a></b>(2),<b><a style="color: #660000;" href="http://linux.die.net/man/1/perlfunc" rel="nofollow">perlfunc</a></b>(1), <b><a style="color: #660000;" href="http://linux.die.net/man/7/pipe" rel="nofollow">pipe</a></b>(7), <b><a style="color: #660000;" href="http://linux.die.net/man/3/pmrecordsetup" rel="nofollow">pmrecordsetup</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/pmtimerecv" rel="nofollow">pmtimerecv</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/2/prctl" rel="nofollow">prctl</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/5/proc" rel="nofollow">proc</a></b>(5), <b><a style="color: #660000;" href="http://linux.die.net/man/3/pth" rel="nofollow">pth</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/7/pty" rel="nofollow">pty</a></b>(7), <b><a style="color: #660000;" href="http://linux.die.net/man/4/random" rel="nofollow">random</a></b>(4), <b><a style="color: #660000;" href="http://linux.die.net/man/3/rpc" rel="nofollow">rpc</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/rpc_soc" rel="nofollow">rpc_soc</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/rpc_svc_calls" rel="nofollow">rpc_svc_calls</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/rpc_svc_reg" rel="nofollow">rpc_svc_reg</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/4/rtc" rel="nofollow">rtc</a></b>(4),<b><a style="color: #660000;" href="http://linux.die.net/man/3/sctp_connectx" rel="nofollow">sctp_connectx</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/session_api" rel="nofollow">session_api</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/shellout" rel="nofollow">shellout</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/2/signalfd" rel="nofollow">signalfd</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/5/slapd-ldap" rel="nofollow">slapd-ldap</a></b>(5), <b><a style="color: #660000;" href="http://linux.die.net/man/5/slapd-meta" rel="nofollow">slapd-meta</a></b>(5), <b><a style="color: #660000;" href="http://linux.die.net/man/3/snmp_agent_api" rel="nofollow">snmp_agent_api</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/snmp_alarm" rel="nofollow">snmp_alarm</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/snmp_api" rel="nofollow">snmp_api</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/snmp_sess_timeout" rel="nofollow">snmp_sess_timeout</a></b>(3),<b><a style="color: #660000;" href="http://linux.die.net/man/2/socket" rel="nofollow">socket</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/7/socket" rel="nofollow">socket</a></b>(7), <b><a style="color: #660000;" href="http://linux.die.net/man/5/ssh-ldap.conf" rel="nofollow">ssh-ldap.conf</a></b>(5), <b><a style="color: #660000;" href="http://linux.die.net/man/5/sssd-ldap" rel="nofollow">sssd-ldap</a></b>(5), <b><a style="color: #660000;" href="http://linux.die.net/man/2/syscalls" rel="nofollow">syscalls</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/7/tcp" rel="nofollow">tcp</a></b>(7), <b><a style="color: #660000;" href="http://linux.die.net/man/2/timerfd_create" rel="nofollow">timerfd_creat</p>
<p>e</a></b>(2), <b><a style="color: #660000;" href="http://linux.die.net/man/4/tty_ioctl" rel="nofollow">tty_ioctl</a></b>(4), <b><a style="color: #660000;" href="http://linux.die.net/man/3/ualarm" rel="nofollow">ualarm</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/7/udp" rel="nofollow">udp</a></b>(7), <b><a style="color: #660000;" href="http://linux.die.net/man/3/usleep" rel="nofollow">usleep</a></b>(3), <b><a style="color: #660000;" href="http://linux.die.net/man/3/vga_waitevent" rel="nofollow">vga_waitevent</a></b>(3),<b><a style="color: #660000;" href="http://linux.die.net/man/1/zshmodules" rel="nofollow">zshmodules</a></b>(1)</p></blockquote>
<h2 id="example-program">实例程序</h2>
<h3 id="input-detect">输入监测</h3>
<p>检测键盘有无输入，完整的程序如下：</p>
<pre class="lang:default decode:true">#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;
#include &lt;string.h&gt;

#include &lt;unistd.h&gt;

#include &lt;sys/select.h&gt;
#include &lt;sys/time.h&gt;
#include &lt;sys/types.h&gt;
#include &lt;sys/socket.h&gt;
#include &lt;arpa/inet.h&gt;

#define BUFLEN 32U

void errpro(int condition, const char *errmsg) {
    if (condition) {
        perror(errmsg);
        exit(EXIT_FAILURE);
    }
}

int main(int argc, char *argv[]) {
    fd_set rdfds;
    struct timeval tv;
    while (1) {
        FD_ZERO(&amp;rdfds);
        FD_SET(STDIN_FILENO, &amp;rdfds);
        tv.tv_sec = 2;
        tv.tv_usec = 0;
        int cnt = select(STDIN_FILENO+1, &amp;rdfds, NULL, NULL, &amp;tv);
        switch (cnt) {
            case -1:
                errpro(EXIT_FAILURE, "select");
                break;
            case 0:
                printf("timeoutn");
                continue;
                break;
            default:
                if (FD_ISSET(STDIN_FILENO, &amp;rdfds)) {
                    char buf[BUFLEN];
                    cnt = read(STDIN_FILENO, buf, BUFLEN-1);
                    errpro(-1 == cnt, "read");
                    if (0 == cnt) // EOF
                        return 0;
                    printf("read: %sn", buf);
                }
                break;
        }
    }
    return 0;
}</pre>
<h3 id="web-server">web 服务器</h3>
<p>利用Select模型，设计的web服务器：</p>
<pre class="lang:c++ decode:true" data-url="https://gist.githubusercontent.com/oxnz/855b94ee4a4aae3c8c88/raw/0378a779e499bd2ab8c7bffa70cbee836202019a/telnet.cpp">#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;
#include &lt;string.h&gt;

#include &lt;unistd.h&gt;

#include &lt;sys/select.h&gt;
#include &lt;sys/time.h&gt;
#include &lt;sys/types.h&gt;
#include &lt;sys/socket.h&gt;
#include &lt;arpa/inet.h&gt;

#define BUFLEN	10U
unsigned char buf[BUFLEN];
const unsigned short PORT(8888);
#define BACKLOG	16U

void errpro(int condition, const char *errmsg) {
	if (condition) {
		perror(errmsg);
		exit(EXIT_FAILURE);
	}
}

int clientpro(int fda[], unsigned int idx) {
	int ret(0);
	int cnt = recv(fda[idx], buf, BUFLEN-1, 0);
	errpro(-1 == cnt, "recv");
	if (0 == cnt) { // client go off
		printf("client[%u] goes offn", idx);
	} else {
		ret = cnt;
		printf("--&gt;[0x%x]n", buf[0]);
		switch (buf[0]) {
			case 0x04:
				printf("^Dn");
				break;
			case 0x73:
				printf("getstatusn");
				break;
			case 0xff:
				printf("client[%u] logoutn", idx);
				return 0;
				break;
			default:
				break;
		}
		buf[cnt] = ' ';
		printf("message from client[%u]:n%s", idx, buf);
		while (BUFLEN-1 == cnt) {
			cnt = recv(fda[idx], buf, BUFLEN-1, 0);
			ret += cnt;
			errpro(-1 == cnt, "recv");
			buf[cnt] = ' ';
			printf("%s", buf);
		}
		char msg[] = "Hello, client xn";
		msg[strlen(msg)-2] = idx + '0';
		errpro(-1 == send(fda[idx], msg, strlen(msg)+1, 0), "send");
	}
	return ret;
}

int main(int argc, char *argv[]) {
	struct sockaddr_in server_addr, client_addr;
	socklen_t sin_size(sizeof(server_addr));
	int sockfd;
	errpro(-1 == (sockfd = socket(AF_INET, SOCK_STREAM, 0)), "socket");
	int opval = 1;
	errpro(-1 == setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &amp;opval,
				sizeof(opval)), "setsockopt");
	server_addr.sin_family = AF_INET;
	server_addr.sin_port = htons(PORT);
	server_addr.sin_addr.s_addr = INADDR_ANY;
	memset(server_addr.sin_zero, 0, sizeof(server_addr.sin_zero));

	errpro(-1 == bind(sockfd, (struct sockaddr *)&amp;server_addr,
				sin_size), "bind");
	errpro(-1 == listen(sockfd, BACKLOG), "listen");
	printf("listen on port: %un", PORT);

	int maxsockfd(sockfd);
	struct timeval tv;
	unsigned int conn_cnt(0);
	int fda[BACKLOG]; // accepted connection fds
	memset(fda, 0, sizeof(fda));
	fd_set fds;
	while (1) {
		// init file descriptor set
		FD_ZERO(&amp;fds);
		FD_SET(sockfd, &amp;fds);
		// set timeout
		tv.tv_sec = 10;
		tv.tv_usec = 0;
		// add active connection to fd set
		for (unsigned int i = 0; i &lt; BACKLOG; ++i) {
			if (fda[i] != 0) {
				FD_SET(fda[i], &amp;fds);
			}
		}
		switch (select(maxsockfd+1, &amp;fds, NULL, NULL, &amp;tv)) {
			case -1:
				errpro(EXIT_FAILURE, "select");
				break;
			case 0:
				printf("timeoutn");
				continue;
				break;
			default:
				break;
		}
		// check every fd in the set
		for (unsigned int i = 0; i &lt; conn_cnt; ++i) {
			if (FD_ISSET(fda[i], &amp;fds)) {
				if (0 == clientpro(fda, i)) {
					--conn_cnt;
					close(fda[i]);
					FD_CLR(fda[i], &amp;fds);
					fda[i] = 0;
				}
			}
		}

		// check whether a new connection comes
		if (FD_ISSET(sockfd, &amp;fds)) {
			int fd = accept(sockfd, (struct sockaddr *)&amp;client_addr, &amp;sin_size);
			errpro(-1 == fd, "accept");

			// add fd to queue
			if (conn_cnt &lt; BACKLOG) {
				fda[conn_cnt++] = fd;
				printf("new connection client[%u] %s:%un", conn_cnt,
						inet_ntoa(client_addr.sin_addr),
						ntohs(client_addr.sin_port));
				maxsockfd = fd &gt; maxsockfd ? fd : maxsockfd;
			} else {
				printf("max connections reached, ignore new connectionn");
				char msg[] = "sorry, server is busy, please try latern";
				errpro(-1 == send(fd, msg, strlen(msg)+1, 0), "send");
				close(fd);
			}
		}
		printf("client count: %un", conn_cnt);
	}
	// close active connections
	for (unsigned int i = 0; i &lt; BACKLOG; ++i) {
		fda[i] != 0 ? close(fda[i]) : 0;
	}

	return 0;
}</pre>
<h2 id="summary">总结</h2>
<p>理解select模型的关键在于理解fd_set,为说明方便，取fd_set长度为1字节，fd_set中的每一bit可以对应一个文件描述符fd。则1字节长的fd_set最大可以对应8个fd。</p>
<ol>
<li>执行fd_set set; FD_ZERO(&amp;set);则set用位表示是0000,0000。</li>
<li>若fd＝5,执行FD_SET(fd,&amp;set);后set变为0001,0000(第5位置为1)</li>
<li>若再加入fd＝2，fd=1,则set变为0001,0011</li>
<li>执行select(6,&amp;set,0,0,0)阻塞等待</li>
<li>若fd=1,fd=2上都发生可读事件，则select返回，此时set变为0000,0011。注意：没有事件发生的fd=5被清空。</li>
</ol>
<p>基于上面的讨论，可以轻松得出select模型的特点：</p>
<ol>
<li>可监控的文件描述符个数取决与sizeof(fd_set)的值。我这边服务 器上sizeof(fd_set)＝512，每bit表示一个文件描述符，则我服务器上支持的最大文件描述符是512*8=4096。据说可调，另有说虽 然可调，但调整上限受于编译内核时的变量值。本人对调整fd_set的大小不太感兴趣，参考<a href="http://www.cppblog.com/CppExplore/archive/2008/03/21/45061.html">技术系列之网络模型（二）</a>中的模型2(1)可以有效突破select可监控的文件描述符上限。</li>
<li>将fd加入select监控集的同时，还要再使用一个数据结构array保存放到select监控集中的fd，一是用于再select 返回后，array作为源数据和fd_set进行FD_ISSET判断。二是select返回后会把以前加入的但并无事件发生的fd清空，则每次开始 select前都要重新从array取得fd逐一加入（FD_ZERO最先），扫描array的同时取得fd最大值maxfd，用于select的第一个 参数。</li>
<li>可见select模型必须在select前循环array（加fd，取maxfd），select返回后循环array（FD_ISSET判断是否有时间发生）。</li>
</ol>
<h2 id="refs">参考</h2>
<ol class="refs">
<li><a href="http://blog.csdn.net/tianmohust/article/details/6595998">http://blog.csdn.net/tianmohust/article/details/6595998</a></li>
<li>http://publib.boulder.ibm.com/infocenter/aix/v7r1/index.jsp?topic=%2Fcom.ibm.aix.progcomm%2Fdoc%2Fprogcomc%2Fsocket_io_md.htm</li>
<li>http://stackoverflow.com/questions/1150635/unix-nonblocking-i-o-o-nonblock-vs-fionbio</li>
</ol>