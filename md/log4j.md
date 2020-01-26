```properties
### 配置根 ###
log4j.rootLogger = debug,console,fileAppender,dailyRollingFile,ROLLING_FILE,MAIL

### 设置输出sql的级别，其中logger后面的内容全部为jar包中所包含的包名 ###
log4j.logger.org.apache=debug
log4j.logger.java.sql.Connection=debug
log4j.logger.java.sql.Statement=debug
log4j.logger.java.sql.PreparedStatement=debug
log4j.logger.java.sql.ResultSet=debug

### 配置输出到控制台 ###
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern =  %d{ABSOLUTE} %5p %c{1}:%L - %m%n

### 配置输出到文件 ###
log4j.appender.fileAppender = org.apache.log4j.FileAppender
log4j.appender.fileAppender.File = logs/log.log
log4j.appender.fileAppender.Append = true
log4j.appender.fileAppender.Threshold = DEBUG
log4j.appender.fileAppender.layout = org.apache.log4j.PatternLayout
log4j.appender.fileAppender.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

### 配置输出到文件，并且每天都创建一个文件 ###
log4j.appender.dailyRollingFile = org.apache.log4j.DailyRollingFileAppender
log4j.appender.dailyRollingFile.File = logs/log.log
log4j.appender.dailyRollingFile.Append = true
log4j.appender.dailyRollingFile.Threshold = DEBUG
log4j.appender.dailyRollingFile.layout = org.apache.log4j.PatternLayout
log4j.appender.dailyRollingFile.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

### 配置输出到文件，且大小到达指定尺寸的时候产生一个新的文件 ###
log4j.appender.ROLLING_FILE=org.apache.log4j.RollingFileAppender
log4j.appender.ROLLING_FILE.Threshold=ERROR
log4j.appender.ROLLING_FILE.File=rolling.log
log4j.appender.ROLLING_FILE.Append=true
log4j.appender.ROLLING_FILE.MaxFileSize=10KB
log4j.appender.ROLLING_FILE.MaxBackupIndex=1
log4j.appender.ROLLING_FILE.layout=org.apache.log4j.PatternLayout
log4j.appender.ROLLING_FILE.layout.ConversionPattern=[framework] %d - %c -%-4r [%t] %-5p %c %x - %m%n

#### 配置输出到邮件 ###
#log4j的邮件发送appender，如果有必要你可以写自己的appender
log4j.appender.MAIL=org.apache.log4j.net.SMTPAppender
#发送邮件的门槛，仅当等于或高于ERROR（比如FATAL）时，邮件才被发送
log4j.appender.MAIL.Threshold=ERROR
#缓存文件大小，日志达到10k时发送Email
log4j.appender.MAIL.BufferSize=10
#发送邮件的邮箱帐号
log4j.appender.MAIL.From=940695836@qq.com
#SMTP邮件发送服务器地址
log4j.appender.MAIL.SMTPHost=smtp.qq.com
#SMTP发送认证的帐号名
log4j.appender.MAIL.SMTPUsername=940695836@qq.com
#SMTP发送认证帐号的密码
log4j.appender.MAIL.SMTPPassword=knkpswudztrrbfif
#是否打印调试信息，如果选true，则会输出和SMTP之间的握手等详细信息
log4j.appender.MAIL.SMTPDebug=false
#邮件主题
log4j.appender.MAIL.Subject=Log4JErrorMessage
#发送到什么邮箱，如果要发送给多个邮箱，则用逗号分隔；
#如果需要发副本给某人，则加入下列行
#log4j.appender.MAIL.Bcc=xxx@xxx.xxx
log4j.appender.MAIL.To=940695836@qq.com
log4j.appender.MAIL.layout=org.apache.log4j.PatternLayout
log4j.appender.MAIL.layout.ConversionPattern=[framework]%d - %c -%-4r[%t]%-5p %c %x -%m%n


```

