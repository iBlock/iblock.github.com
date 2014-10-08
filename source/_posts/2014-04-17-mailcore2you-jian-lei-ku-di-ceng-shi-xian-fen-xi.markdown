---
published: TRUE
layout: post
title: "MailCore2邮件类库底层实现分析"
date: 2014-04-17 11:58:15 +0800
comments: true
categories: [MailCore2,LibEtPan,邮件库]

---

##MailCore2邮件库概述
MailCore是一个Mac和iOS下的email库。使用它能轻易发送email，支持SMTP, IMAP, POP3以及RFC822。现在，来自Sparrow和MailCore 1.0的开发者们打造了新的MailCore--MailCore 2!
 
具体功能如下：

*	POP, IMAP, and SMTP support
*	RFC822 parser and generator
*	UI widgets for rendering HTML messages
*	Asynchronous APIs with Objective-C blocks
*	iOS and Mac support
*	Portable core engine in C++

<!--more-->

##MailCore2邮件库主体

![alt text](/images/2014/04/17/MailCore2.png "MailCore2")

从上图可以看出，LibEtPan这个库才是最核心的库，所有邮件的相关实现功能都是在该库下实现的，MailCore2是C++和Objective-C的混合项目，它先是用C++对底层的LibEtPan作了相对较全面的面向对象封装，这部分C++的封装与LibEtPan一样可以用于Linux和Windows平台上的应用，然后再通过Objective-C继续对C++的这一层进行抽象、封装后供Mac和IOS应用使用。

##LibEtPan
###LibEtPan分析
####概述
LibEtPan是一个处理对邮箱的各种访问需求的库，这个库对IMAP、POP3、SMTP、MBOX、MH等都有良好支持。它采用C语言编写，但可以方便地在C++环境下编译，它是一个跨平台的库，主要运行在MAC、UNIX平台上，这个库在非Windows平台上的表现应该会比Windows平台上要好。
使用这个库有两种方式：

*	直接使用底层的函数，这样访问不同的对象时接口也不完全一样，但是代码非常直接，容易读。
*	通过IMAP、POP3等各自的driver使用上层封装好了的函数，通过函数指针的形式使得各种方式下的访问接口是一样的，灵活性很高，但是代码读起来就不那么直接了。另外，LibEtPan的上层封装集成了自己实现的缓存和多线程功能。

这个库对相关邮件规范的支持情况如下：

*	IMAP	
LibEtPan实现了RFC 2060 VERSION 4rev1定义了的每一条指令。
*	POP3	
RFC 1939 以及 RFC2449 规定的每一个指令，LibEtPan都进行了实现。RFC 1734 的POP3 认证指令没有实现。
*	SMTP	
RFC 2821 和 RFC 1891 规定的每一条指令LibetPan都进行了实现。

LibetPan对其源代码文件进行了规范和一致的命名，并且保证每个源代码文件的内容与其名字所提示的作用一致，结合其对IMAP的实现，具体而言：

*	mailimap.[ch]	
包括实现IMAP相关 RFC 指令的代码。
*	mailimap_helper.[ch]	
为上述函数提供的helper接口。
*	mailmap_types.[ch]	
包括IMAP实现所需要的类型以及类型的构建函数。
*	mailimap_types_helper.[ch]	
该文件包含一组函数，用来创建使用 IMAP 实现模块时所必须的数据。
*	mailimap_socket.[ch]	
包含通过 TCP 连接 IMAP 服务器的功能相关的函数。
*	mailimap_ssl.[ch]	
提供了通过 TLS 层连接 IMAP 服务器的函数。

####LibEtPan结构说明
{% blockquote %}
库 libetpan 的内容庞大,从对邮件的支持上看,从通过 Socket 建立到服务器 的 TCP 连接、到解析邮件的每个字符、到对各种格式和协议的处理,直至针对客 户端应用的封装,libetpan 都做了周到的工作;从对库的封装上看,它不仅提供 了需求对应的功能,还按基础功能->会话->存储这样三个层次对整个库进行了设 计和封装,使得逻辑清晰,有助于调用方的理解和使用。其中,基础功能层以底 层 C 函数的形式提供了对各邮件协议的具体支持;会话层对底层封装了对远程邮 件服务器或者本地文件系统的连接并且维护相关的会话,对上层则利用对底层连 接的封装,按上层(存储/文件夹层)的功能的需要,提供了相应的函数,使得 上层不必考虑连接、会话方面的内容;存储/文件夹层一方面从应用层取得认证 信息并传递到会话层由其自动获取和维护连接,另外则专注于实现邮件存储、文 件夹部分的业务逻辑。
{% endblockquote %}

这个库的完整架构基本是下图这样子,从上到下依次可划分为应用层、存储/文件夹层、会话层、邮件协议支持、 邮件格式支持这样几个层次,通过专门设计的数据结构、函数指针等把各层关联了起来：

![alt text](/images/2014/04/17/LibEtPan.png "LibEtPan")

对应上图中的各层,libetpan 分别对存储/文件夹、会话和邮件提供了统称为 driver 的封装工具,隐藏了底层细节,供上层调用。

###LibEtPan库各部分功能说明
####IMF/MIME
IMF：Internet Message Format	
MIME：Multipurpose Internet Mail Extensions

库 libetpan 通过这 2 个模块实现了对邮件中从小到具体字符、大到邮件结构数据的解析,libetpan 还使用这里定义的各种数据结构来封装邮件各部分的数据

*	IMF部分源代码作用说明	
![alt text](/images/2014/04/17/IMF.png "IMF")
*	MIME部分源代码作用说明	
![alt text](/images/2014/04/17/MIME.png "MIME")

libetpan 在取得例如邮件内容这样的文本时,内部进行了良好的缓冲、数据 流处理,从而能够稳定地处理例如邮件内容这样的大数据,避免突然占用大量内 存等引起的异常情况的发生,下面以通过 pop3 方式获取邮件内容功能的实现予以说明:

{% blockquote %}
在最底层(mailstream_socket.c)通过系统提供的 read 函数读取在socket 通信 的响应,这部分的工作是 OS 完成的,libetpan 只是调用了这个函数把 socket 请 求的响应读到一个 char*中,这个内容然后被放到 mailstream 的 char * read_buffer 成员中,然后在 mailstream.c 中一边把 read_buffer 中的内容直接内存拷贝到 MMAPString *中,一边把新读取到的内容调整、填充到 read_buffer,最后把响 应结果放到了 mailpop3*的 MMAPString * pop3_response_buffer 成员中。在从 mailpop3*中实际获取邮件内容时,也是先把内容逐步读取到一个MMAPString*, 最后在真正需要的时候才读取到 char*中。至于MMAPString,它的实现包含了大量对内存的操作和系统调用,从使用的角度,可以不用深入考虑其细节,只要把它当作一个数据不断增加时容量能自 动增加的字符串即可。
{% endblockquote %}

####对邮件协议的支持
*	IMAP	
主要的相关文件及其作用参见 2.1.1 节的说明。
核心数据结构总共有 3 个结构体:mailimap、imap_mailstorage 以及imap_mailstorage_driver。
Mailimap 定义在在 imapdriver_types.h 中,它的主要成员是关于操作请求的响应、认证和查询当前进度的函数指针这三类：	
{% codeblock lang:c++ %}
struct mailimap {char * imap_response;/* internals */mailstream * imap_stream; 
size_t imap_progr_rate;progress_function * imap_progr_fun; 
MMAPString * imap_stream_buffer;MMAPString * imap_response_buffer; 
int imap_state;int imap_tag;struct mailimap_connection_info * imap_connection_info;	
struct mailimap_selection_info * imap_selection_info;	
struct mailimap_response_info * imap_response_info;	
struct {void * sasl_conn;const char * sasl_server_fqdn; 
const char * sasl_login;const char * sasl_auth_name; 
const char * sasl_password; 
const char * sasl_realm;void * sasl_secret;} imap_sasl;
time_t imap_idle_timestamp; 
time_t imap_idle_maxdelay;mailprogress_function * imap_body_progress_fun; mailprogress_function * imap_items_progress_fun; 
void * imap_progress_context;};
{% endcodeblock %}

imap_mailstorage 在 imapdriver_types.h 中定义了连接 IMAP 服务器的过程中 认证过程需要的数据：	
{% codeblock lang:c++ %}
struct imap_mailstorage { 
char * imap_servername; 
uint16_t imap_port;char * imap_command; 
int imap_connection_type;int imap_auth_type;char * imap_login; /* deprecated */char * imap_password; /* deprecated */int imap_cached;char * imap_cache_directory;
struct {int sasl_enabled;char * sasl_auth_type;char * sasl_server_fqdn; 
char * sasl_local_ip_port; 
char * sasl_remote_ip_port; 
char * sasl_login;char * sasl_auth_name; 
char * sasl_password;char * sasl_realm;} imap_sasl;
char * imap_local_address;uint16_t imap_local_port; };
{% endcodeblock %}

imap_mailstorage_driver 在 imapstorage.c 中定义了符合统一接口 mailstorage_driver 的要求的一组函数指针：	

{% codeblock lang:c++ %}
static mailstorage_driver imap_mailstorage_driver = {￼/* sto_name */ "imap",/* sto_connect */ imap_mailstorage_connect,/* sto_get_folder_session */imap_mailstorage_get_folder_session, 
/* sto_uninitialize */ imap_mailstorage_uninitialize};
{% endcodeblock %}

*	POP3

![alt text](/images/2014/04/17/POP3.png "POP3")

主要的数据结构如下,它的主要成员包括用户认证、对操作请求的响应和结 果的封装两类：

{% codeblock lang:c++ %}
struct mailpop3 {char * pop3_response; 
char * pop3_timestamp;/* internals */mailstream * pop3_stream;size_t pop3_progr_rate; 
progress_function * pop3_progr_fun;MMAPString * pop3_stream_buffer; 
MMAPString * pop3_response_buffer;carray * pop3_msg_tab; 
int pop3_state;unsigned int pop3_deleted_count;
struct {void * sasl_conn;const char * sasl_server_fqdn; 
const char * sasl_login;const char * sasl_auth_name; 
const char * sasl_password; 
const char * sasl_realm;void * sasl_secret;} pop3_sasl;

};
{% endcodeblock %}

*	SMTP

![alt text](/images/2014/04/17/SMTP.png "SMTP")

除了公共的数据结构 mailstream 外,对 SMTP 的支持基本上只使用了下面的数据结构：

{% codeblock lang:c++ %}
struct mailsmtp {mailstream * stream;size_t progr_rate; 
progress_function * progr_fun;char * response;MMAPString * line_buffer; 
MMAPString * response_buffer;int esmtp; /* contains flags MAILSMTP_ESMTP_* */ 
int auth; /* contains flags MAILSMTP_AUTH_* */
struct {void * sasl_conn;const char * sasl_server_fqdn; 
const char * sasl_login;const char * sasl_auth_name; 
const char * sasl_password; 
const char * sasl_realm;void * sasl_secret;} smtp_sasl;
size_t smtp_max_msg_size; 
mailprogress_function * smtp_progress_fun; 
void * smtp_progress_context;int response_code; 
};
{% endcodeblock %}

发送邮件的响应,由 mailstream.c 负责读取并读取的结果存储进 MMAPString * line_buffer。

####会话层
libetpan 主要针对 IMAP、POP3、NNTP、MH、FEED、MBOX 提供了会话及对 其操作的封装(称为 driver)的支持,我们大量使用的只有对 IMAP 的部分。

会话层可以访问本地文件系统或者远程邮件服务器,可以通过它直接发送邮 件操作命令到邮件服务器,邮件存储使用会话与服务器通信;邮件文件夹也使用 会话从服务器获取信息或者向服务器发送信息,按具体实现的不同,整个过程中 可以使用统一会话,也可以使用不同的会话。

会话在 libetpan 中使用结构体 mailsession 来表示：

{% codeblock lang:c++ %}
struct mailsession {void * sess_data; 
mailsession_driver * sess_driver;};
{% endcodeblock %}

一个会话有一个 driver,driver 中的内容具体封装了会话相关的操作,driver 提供的信息包括：	

*	name:driver 的名字
*	initialize():调用mailsession_new()创建会话时会调用这个函数,它会创建一个特定于 driver 的数据结构并存储在会话中,这个数据结果对于不同的实现(例如 IMAP、NNTP 等)具体内容不同,也作为会话状态使用。
*	uninitialize():释放调用 initialize()时创建的数据结构。
*	parameters():实现特定于给定邮件访问的函数。
*	connect_stream():把数据流连接到会话。
*	connect_path():通知主路径到会话。
*	starttls():把当前流转换一个 TLS 流。
*	login():使用用户名和密码来认证会话。
*	logout():退出会话并且关闭流。
*	noop():不作任何操作,但不断轮询到服务器的连接的状态,相当于心跳保持。
*	check_folder():通过向服务器发送一些指令的方式来对会话做一次检查。
*	select_folder():选择一个邮件文件夹(mailbox)。
*	expunge_folder():删除所有标记为\Deleted 的邮件。
*	status_folder():查询文件夹的状态(包括邮件总数、最近的邮件数、未读的邮件数)。
*	append_message():把一个符合 RFC 2822 要求的邮件加入到当前文件夹。
*	get_messages_list():返回当前文件夹中所有邮件的列表。
*	get_envelopes_list():取得解析了的 envelope 字段。
*	remove_message():彻底删除当前文件夹中给定的邮件。
*	get_message:以 mailmessage 结构体的形式返回给定序号的邮件。

在 IMAP 的情况下就是一个例子,相应的 driver 的创建或者构造其实很简单, 直接使用函数指针进行结构体赋值,这样就把一组与会话相关的函数通过一个结 构体封装了起来,从而可以用相同的代码调用,间接实现了 CPP Template 的效果, 而且在接下来可以直接通过相应的变量去使用：

{% codeblock lang:c++ %}
static mailsession_driver local_imap_session_driver = { 
/* sess_name */ "imap",/* sess_initialize */ imapdriver_initialize,/* sess_uninitialize */ imapdriver_uninitialize,/* sess_parameters */ imapdriver_parameters,/* sess_connect_stream */ imapdriver_connect_stream, 
/* sess_connect_path */ NULL,/* sess_starttls */ imapdriver_starttls,/* sess_login */ imapdriver_login,/* sess_logout */ imapdriver_logout, 
/* sess_noop */ imapdriver_noop,/* sess_build_folder_name */imapdriver_build_folder_name, 
/* sess_create_folder */ imapdriver_create_folder,/* sess_delete_folder */ imapdriver_delete_folder,/* sess_rename_folder */ imapdriver_rename_folder,/* sess_check_folder */ imapdriver_check_folder,/* sess_examine_folder */ imapdriver_examine_folder, 
/* sess_select_folder */ imapdriver_select_folder,/* sess_expunge_folder */ imapdriver_expunge_folder,
/* sess_status_folder */ imapdriver_status_folder,/* sess_messages_number */ imapdriver_messages_number, 
/* sess_recent_number */ imapdriver_recent_number,/* sess_unseen_number */ imapdriver_unseen_number,/* sess_list_folders */ imapdriver_list_folders,/* sess_lsub_folders */ imapdriver_lsub_folders,/* sess_subscribe_folder */ imapdriver_subscribe_folder,/* sess_unsubscribe_folder */imapdriver_unsubscribe_folder,
/* sess_append_message */ imapdriver_append_message,/* sess_append_message_flags */ imapdriver_append_message_flags, 
/* sess_copy_message */ imapdriver_copy_message,/* sess_move_message */ NULL,/* sess_get_message */ imapdriver_get_message,/* sess_get_message_by_uid */imapdriver_get_message_by_uid,/* sess_get_messages_list */imapdriver_get_messages_list, 
/* sess_get_envelopes_list */imapdriver_get_envelopes_list, 
/* sess_remove_message */ imapdriver_remove_message,/* sess_login_sasl */ imapdriver_login_sasl 
};
{% endcodeblock %}

libetpan 对具体的邮件使用下面的结构体封装,除了包含邮件本身的消息外, 对于具体的 mailmessage 实例可以获取相应的会话,还可以获取对邮件的处理的 driver：

{% codeblock lang:c++ %}
struct mailmessage {mailsession * msg_session; 
mailmessage_driver * msg_driver; 
uint32_t msg_index;char * msg_uid;size_t msg_size;// export internal_date; add by liamstruct mailimap_date_time * internal_date; 
struct mailimf_fields * msg_fields;struct mail_flags * msg_flags; 
int msg_resolved;
struct mailimf_single_fields msg_single_fields; 
struct mailmime * msg_mime;
/* internal data */int msg_cached; 
void * msg_data;
/*msg_folder field :used to reference the mailfolder, this is a workaround due to the problem with initial conception, where folder notion did not exist.*/
void * msg_folder;/* user data */void * msg_user_data;};
{% endcodeblock %}

通过会话,可以取得邮件,libetpan 使用结构体 mailmessage 表示一封邮件。 与对会话的处理相似,在处理邮件时,libetpan 使用了类似的方式:通过 mailmessage_driver 提供了一组操作邮件内容的函数,具体包括(一个邮件可以 包含数量不限的 MIME 部分,MIME 之间可嵌套,每个 MIME 包含消息头和消息 体两部分)：

*	name:driver 的名字
*	initialize():创建以-分割的邮件 uid 并保存在 mailmessage*中,在调用mailmessage_init()时会调用这个方法。
*	uninitialize():释放 initialize()方法创建的数据,mailmessage_free()会调用这个方法。
*	flush():从内存中释放所有临时的邮件结构体,例如 MIME 消息的结构体。
*	fetch_result_free():释放所有以 fetch 开头的方法返回的字符串。
*	fetch():返回邮件的内容,包括头、文本。
*	fetch_header():返回 header 部分的内容。
*	fetch_body():返回邮件的不包含 header 部分内容的正文。
*	fetch_size():取得邮件的大小。
*	get_bodystructure():取得邮件的 MIME 结构信息,就是通常说的bodystructure 的内容。
*	fetch_section():取得给定的 MIME 部分的内容。
*	fetch_section_header():返回给定 MIME 部分的消息的消息头。
*	fetch_section_mime():返回给定 MIME 部分的 MIME 头。
*	fetch_section_body():返回给定 MIME 部分文本,对一个 MIME 消息则返回不包含头的消息内容。
*	fetch_envelope():返回一个 mailimf_fields 结构体,其中包含一组有相应driver 选取的 mailimf_field;mailimf_field 是一个结构体,其中包含有一个公用体,通过 int 类型的 fld_type 指明了公用体的实际内容。
*	get_flags():返回一个与该邮件有关的标志。在直接使用 message->flags获取标志前须确保调用了这个方法。

mailmessage_driver 也通过结构体封装了这些函数的指针:

{% codeblock lang:c++ %}
static mailmessage_driver local_imap_message_driver = { 
/* msg_name */ "imap",/* msg_initialize */ imap_initialize, 
/* msg_uninitialize */ NULL,/* msg_flush */ imap_flush, 
/* msg_check */ imap_check,/* msg_fetch_result_free */ imap_fetch_result_free,/* msg_fetch */ imap_fetch,/* msg_fetch_header */ imap_fetch_header,/* msg_fetch_body */ imap_fetch_body,/* msg_fetch_size */ imap_fetch_size,/* msg_get_bodystructure */ imap_get_bodystructure,/* msg_fetch_section */ imap_fetch_section,/* msg_fetch_section_header */imap_fetch_section_header, 
/* msg_fetch_section_mime */ imap_fetch_section_mime, 
/* msg_fetch_section_body */ imap_fetch_section_body,/* msg_fetch_envelope */ imap_fetch_envelope,/* msg_get_flags */ imap_get_flags
};
{% endcodeblock %}

####邮件存储/文件夹层
一个邮件存储(结构体 mailstorage)实例代表一个邮件服务器或者一个主路径,它实际上可以是一个 IMAP 服务器,一个 MH 或 mbox 文件的根路径。

{% codeblock lang:c++ %}
struct mailstorage {char * sto_id;void * sto_data;mailsession * sto_session;mailstorage_driver * sto_driver;clist * sto_shared_folders; /* list of (struct mailfolder *) */void * sto_user_data; };
{% endcodeblock %}

获得邮件存储实例后,可以根据它在服务器上创建一个文件夹(通过结构体 mailfolder 表示),文件夹通常还被称作 mailbox,也就是收件箱、发件箱、草稿 箱等,这个文件夹可以是主路径的子文件夹,通过它可以执行对文件夹有关的操 作。

对 IMAP 而言,文件夹是 IMAP 的 mailbox,对 MH 而言是 MH 存储的其中一 个文件夹,对 MBOX 而言,只有一个文件夹,也就是 MBOX 文件。

从业务逻辑上看,mailbox 像是一个房子,但是房子里面可以有数量不定的 房间,在 libetpan 内部,房间就是 mailfolder,但在具体的技术实现上,mailbox 和 mail folder 都统一为 mail folder,用一个结构体表示,该结构体可嵌套,从而 支持 mailbox 和 mail folder 之间的逻辑关系。

文件夹的具体定义如下:

{% codeblock lang:c++ %}
struct mailfolder {char * fld_pathname; 
char * fld_virtual_name;struct mailstorage * fld_storage;mailsession * fld_session; 
int fld_shared_session;
clistiter * fld_pos;struct mailfolder * fld_parent;unsigned int fld_sibling_index;carray * fld_children; /* array of (struct mailfolder *) */void * fld_user_data; };
{% endcodeblock %}

libetpan 对访问邮件的需求提供了三个 driver,上面说明了用于会话和邮件 的 2 个 driver,还有一个用于访问存储/文件夹,访问文件夹是仅需要使用会话的 driver。结构体 mailstorage_driver 定义了 libetpan 在存储/文件夹的访问上支持的 操作,它的定义如下:

{% codeblock lang:c++ %}
struct mailstorage_driver {char * sto_name;int (* sto_connect)(struct mailstorage * storage);int (* sto_get_folder_session)(struct mailstorage * storage,char * pathname, mailsession ** result);void (* sto_uninitialize)(struct mailstorage * storage);};
{% endcodeblock %}

其中:

*	name 是 driver 的名字
*	connect()方法把存储连接到远程服务器或者本地文件系统的路径
*	get_folder_session()方法有两种行为:1)创建一个新的会话并且使之独立于存储使用的会话,然后选择给定的邮箱;2)选择当前存储的会话及其特定的邮箱。
*	uninitialized()方法释放构造 mailstorage 时创建的数据。

在 IMAP、POP3 等操作环境下,mailstorage_driver 会分别被实例化为 imap_mailstorage_driver 和 pop3_mailstorage_driver。

####数据库、缓存文件
libetpan 支持把邮件数据存储到 Berkeley DB 里,如果要使用这个数据库,需 要在编译时进行相应的配置,而且对数据库的支持目前仅限于*nix、Mac 平台。

直接在 libetpan 把邮件数据保存到数据库中,这样做的运行效率会比 sqlite 要高不少,但势必要把一些业务逻辑上的东西带入到 libetpan 中,而且要比通过 Objective C 操作 sqlite 的开发及调优的难度会有一定的增加。

libetpan 的缓存实际就是文件,这些文件可以根据需要使用不同的文件夹包 含起来,在*nix 平台上,通过 mmap 系统调用并以流操作的形式+一定的直接的 内存操作来处理缓存数据,数据相当于缓存到了内存中,倒是真正有缓存的含义。

该缓存支持直接把 mailmessage 等结构体的数据存储到缓冲中。

在 windows 平台上,这个库并没有把 mmap 调用替换成 windows 操作系统 对应的机制, Windows 平台下对该缓存应该还没有真正完成。

####IMAP 实现对邮件搜索功能的支持
Libetpan 对 IMAP 的实现支持按下面定义的数据结构从服务器端对邮件各个 部分的内容进行搜索,但肯定需要服务器端的支持；

{% codeblock lang:c++ %}
struct mailimap_search_key { 
int sk_type;union {char * sk_bcc;struct mailimap_date * sk_before; 
char * sk_body;char * sk_cc;char * sk_from;char * sk_keyword;struct mailimap_date * sk_on; 
struct mailimap_date * sk_since; 
char * sk_subject;char * sk_text;char * sk_to;char * sk_unkeyword;
struct {char * sk_header_name;char * sk_header_value;} sk_header;
uint32_t sk_larger;struct mailimap_search_key * sk_not; 
struct {
	struct mailimap_search_key * sk_or1;	struct mailimap_search_key * sk_or2; 
} sk_or;
struct mailimap_date * sk_sentbefore; 
struct mailimap_date * sk_senton; 
struct mailimap_date * sk_sentsince; 
uint32_t sk_smaller;struct mailimap_set * sk_uid;struct mailimap_set * sk_set;clist * sk_multiple; /* list of (struct mailimap_search_key *) */} sk_data; 
};
{% endcodeblock %}

####其它
libetpan 的一个有意思的地方是以 C 语言实现了一批实用的数据结构,实现了类似相应的 C++模板的效果,使用方便而且编译速度要快很多：

![alt text](/images/2014/04/17/Other.png "Other")

特别是前几个数据结构,在 libetpan、MailCore 里面都大量使用,不管是在 哪一个层次。

libetpan 对上层提供的接口及其作用的说明:

![alt text](/images/2014/04/17/Other2.png "Other")

####总结
Libetpan 虽然采用 C 开发,但由于它尝试用存储->会话->邮件这样一个统一 的框架并以函数指针的方式来封装获取、解析邮件各部分内容的全过程,所以无 法直接从代码之间的调用关系来了解整个过程,必须从各个层次的封装接口入手 才能连接整个过程。

libetpan 对邮件协议的封装中,最复杂的是 IMAP 部分。这里对 IMAP 的封装 接口以下图进行说明:

![alt text](/images/2014/04/17/LibEtPan_IMAP.png "LibEtPan_IMAP")

感谢原文提供者。