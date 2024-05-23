- The application must do most of the work [[1](https://iang.org/ssl/h5_security_begins_at_the_application_and_ends_at_the_mind.html#ref_1)]. For really secure systems, only application security is worthwhile. That is simply because most applications are delivered into environments where they have little control or say over how the underlying system is set up.
  
  **#5.1 Security below the application layer is unreliable**
  
  For security needs, there is little point in for example relying on (specifying) IPSec or the authentication capabilities or PKI or trusted rings or such devices. These things are fine under laboratory conditions, but you have no control once it leaves your door. Out there in the real world, and even within your own development process, there is just too much scope for people to forget or re-implement parts that will break your model.
  
  If you need it, you have to do it yourself. This applies as much to retries and replay protection as to authentication and encryption; in a full secure system you will find yourself dealing with all these issues at the high layer eventually, anyway, so build them in from the start.
  
  Try these quick quizzes. It helps if you close your eyes and think of the question at the deeper levels.
- Is your firewall switched on? How do you know? Or, how do you know when it is not switched on?
- Likewise, is your VPN switched on? How do you know?
- Does TLS provide replay protection [[2](https://iang.org/ssl/h5_security_begins_at_the_application_and_ends_at_the_mind.html#ref_2)]?
- Is wireless access point you are using encrypting [[3](https://iang.org/ssl/h5_security_begins_at_the_application_and_ends_at_the_mind.html#ref_3)]?
  
  These questions lead to some deeper principles.