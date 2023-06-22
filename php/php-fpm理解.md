# PHP-FPM理解

## CGI

CGI(Common Gateway interface)是web服务与外部程序数据交互的一种规范，这些实现了CGI规范的外部程序就称为CGI程序，通常是某些脚本语言的解释器或者可执行文件，通过实现这种规范，可以扩展web的功能。

## FastCGI

每当web服务收到一个网络请求，该请求需要通过CGI规范与外部的CGI程序进行交互才能得到正确的响应，从而返回给请求方。没当这种请求到来时，都需要fork一个CGI程序(fork-and-excute)，这需要花费比较多的时间，这很大程度上限制了服务的性能。为了解决这个问题，提出了fastCGI，这是一个常驻内存的二进制程序，维护着数十个CGI程序，伴随着web服务启动。fastCGI自身不处理请求，而是将请求转发给它维护的CGI程序，由于这些CGI程序都在内存中，不需要fork操作，因此效率会比fork-and-excute的模式效率要高。但是由于提前将CGI程序加载到内存中，fastCGI占中的内存会比较高。

> FastCGI类似于java中的线程池，其维护的CGI程序就相当于线程池中的线程，用完放回到池中，不直接释放回系统。

## PHP-CGI

PHP-CGI就是CGI规范的PHP实现，其实指的就是PHP解释器。

## PHP-FPM

PHP-FPM就是fastCGI的实现。
