---
layout: post
title: "log4jna: The description for Event ID 4096 from source log4jna cannot be found."
redirect_from: "/log4jna-the-description-for-event-id-4096-from-source-log4jna-cannot-be-found/"
date: 2010-10-14 12:25:48
tags: [log4j, jna, win32]
comments: true
dblog_post_id: 130
---
This is a pretty common problem with NTEventLogAppender (both log4j and [log4jna](https://github.com/dblock/log4jna) versions) and generally with firing events to the Windows event log. The events are properly fired, but the text in the event log contains the annoying _"the description for Event ID 4096 from source cannot be found"_ error.

![]({{ site.url }}/images/posts/2010/2010-10-14-log4jna-the-description-for-event-id-4096-from-source-log4jna-cannot-be-found/image_16.jpg)

The Windows event log uses pre-configured sources to render messages. In order to support localization the message format and sometimes the entire message text is external to the actual message and is contained in resource DLLs. There’s a number of great articles that explain, in detail, how this works.

- [CodeProject: Using mc.exe, message resources and the NT event log in your own projects](https://www.codeproject.com/Articles/4166/Using-MC-exe-message-resources-and-the-NT-event-lo)
- [MSDN: Creating a resource DLL](https://msdn.microsoft.com/en-us/library/ms853727.aspx)

Both log4j and Log4jna NTEventLogAppenders ship with a simple message resource DLL. Unlike log4j’s version which contains both code to fire event log events, the log4jna version is a pure message-only resource DLL. It does not need to be on PATH and can therefore be placed in any location on your machine, usually with your application.

For a _demo_ event source we must tell Windows where the resource DLL is by creating an _EventMessageFile_ registry key in the event source (_HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\eventlog\Application\demo_). The value for the key is the full path to _Win32EventLogAppender.dll_.

![]({{ site.url }}/images/posts/2010/2010-10-14-log4jna-the-description-for-event-id-4096-from-source-log4jna-cannot-be-found/image_9.jpg)

Our resource DLL simply says to format every message "as-is".

```
MessageId=4096
Language=English
%1
```

The message renders properly after reopening the event viewer.

![]({{ site.url }}/images/posts/2010/2010-10-14-log4jna-the-description-for-event-id-4096-from-source-log4jna-cannot-be-found/image_14.jpg)
