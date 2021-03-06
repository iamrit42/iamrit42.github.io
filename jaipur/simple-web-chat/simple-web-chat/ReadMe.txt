Its a simple web based chat system using HTML5 websockets / HTML5 SSE with ajax long polling as fall-back (can work with or without any database server)

It can be used as standalone or as module / plugin in any website. Implemented in core php and js code using jquery. Very simple, feature rich and fully customizable chat system. Auto fall-back from html5 websockets to html5 sse to ajax long polling

Fetaures:
1) Registration, login, forgot password
2) Search and add contacts, manage groups
3) Broadcasting, one to one & group chat
4) Desktop notifications, sound alert, auto scroll to new message
5) File attachments
6) Multiple tabbed chat
7) History of old chat messages
& *Audio-Video chat using WebRTC integrated into code, but not yet tested

Separation message write process from server process to improve performance.

All these managed without use of any database server. Its fully standalone but can be easily integrated with any database server using simple cron. Code for db integration is not included in this package.



Overview on structure of data files:

Chat messages are stored inside /tmp folder as temporary files. One message in one file.
Chat History is stored in /h folder.
Files uploaded goes into /pub fodler.
So, make sure you have write permission to this folders from web server user i.e. generally 'www-data'.

Let say user1 is chating with user2,
	files for those messages will be inside /tmp/user1-user2/
	files for broadcasted messages will be inside /tmp/mt/
	folders starting with 'g~' are for groups and will contain files of group chat messages
	once a message file for e.g 'file123' is read by 'user1' to prevent re-reading one file named 'user1_file123' is created
	while reading message files for e.g 'file123', if file 'user1_file123' exists then 'file123' is skiped

If using db server, you need to create a cron which will read all these message files and store it with appropriate relation in db.
Also, if using db server for better performance you can remove or comment deleting and history code and manage it in cron.

History files are created concatinating two user's names or username with group (in case of history for a group)
So, for group each user have separate history files.
History files are stored inside /h for broad casted messages and inside /h/uh for inidividual and group chats

Other files and folders used:
/files/un	User's profile file containing data of contact names, group names, username, email, password, contact and group request, verification code
/files/grp	Contains a file for each group which contains contact's names and requested contacts
/files/ou	Contains a file for each online user and can be used for storing current logged-in session values
/files/um	Contains folder for each relation of user inside that user's folder
			For e.g.
				/files/um/user1 	Will contain a folder 'user1-user2' if user1 has a contact user2 and in case of group it will have folder with 'g~groupname-groupid'.
								This is used to easily retrive chat data from same folder inside /tmp and avoid exessive relation mapping or reading from files.
								So, if user1 and user2 are contacts and are in a 'group1' they both will have once folder in /files/um and inside each folder they will have 'user1-user2' and 'g~group1-123' folders.
									Also, same 'user1-user2' and 'g~group1-123' folders are used in /tmp to store respective message files.

While fetching messages for user1, name of all folders inside /files/um/user1 are obtained and files inside same named folders from /tmp are read and send to user1.

So at server side instead of R-DBMS server file system is constantly monitored for each user, for any new or unread files and as soon as any such file found contents of those files are read and send to the respective user & also that flag file is created so that same file is not fetched again.

Now we need a cron which will read all these message files and store it with appropriate relation in db and once data from a file is inserted delete that temporary message file.

To insert message data in to R-DBMS server we can use a cron at every 1 minute which will read all message files and create separate sql file per each cron and store that sql file in a secure folder and then delete those message files which are read. Now, those sql files can be dumped into mysql at every 30 minutes. The reason behind separating insertion into two crons is to reduce insert frequency into DB server and also at the same time maintain less files on file system to look into for new messages.

In this manner persistent storage of our data is done in R-DBMS server and whenever user needs to show history of chat data can be fetched from R-DBMS. Again, we can limit data in R-DBMS server to let say 3 months for each user after which a download facility can be given for user to get their old data in a file which once downloaded will be deleted after certain period of time and as those data are already deleted from DB server, won't appear in chat history in application.

Thus, excessive sql quering, relation mapping and file reading is avioded. Also creation of huge file is avoided here by creating separate files for each messages and handling as much as possible checks and relations using directory exists and file exists method.

If want to use Websockets, to start web-server for websocket requests, execute  below command in command line:
sudo -u www-data /usr/bin/php -q chat.php

index.php is the entry script so main url of application will be like: http:// ... /simple-web-chat/index.php

To, know more about theoretical (technical) concept in detail you can visit:
http://pls-e.in/project/theory/filesystem_based_data_management_architecture

Attention:
It has been found that if this system is hosted on windows few features won't work, reason is explained below:

1) Naming convention used for group files is not supported on windows, so windows users need to change some other separator in group files instead of ':' symbol.

2) Also grep command is used to search contacts (contact files), so again windows users will have to change this functionality.

Will soon update package for windows compatibility, but till that time if want to use this chat system on windows users will have to modify this two features on their own.
