## web_buffer, a relay server to stop and start the request firehose
Web_buffer is a simple Sinatra Ruby server which operates in two modes. When it is operating in pass-through mode, it simply redirects all incoming POSTs and GETs to configuration-defined server Z.  But when it is in buffering mode, it logs all the requests and returns a successful HTTP status to the clients. There is a companion script executed from the command line which can at any time consume the logged requests and execute them against server Z.

The use case is a situation where server Z is not clustered and must be taken down for maintenance. If it is not crucial that its requests be executed in real time, one can avoid a gap in service by

1.) Initially changing DNS to run all requests to server Z through the web_buffer host
2.) Just prior to taking server Z down, change web_buffer to buffering mode
3.) Perform maintenance on server Z
4.) Bring server Z back up
5.) Switch web_buffer back to pass-through mode

The one wrinkle is that when requests have been saved, web_buffer will wait for all requests on this to be processed before returning to pass-through behavior. The idea here is that there may be an ordering dependency on the requests such that if server Z processes them out of order, there could be trouble. So it is careful to execute the requests that were logged on disk in the order that they were received, and then only once the queue is exhausted, return to the true pass-through mode where requests are redirected to server Z.
