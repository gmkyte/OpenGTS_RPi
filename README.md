# OpenGTS_RPi
/ Installing OpenGTS on a Raspberry Pi 3
/ ======================================
/ 
/ After quite a few attempts, I have installed and tested OpenGTS onto a Rapsberry Pi 3.
/ So, below is a complete installation method all done from the command line via an SSH connection
/ using Putty.exe. The Raspberry Pi was initially set up using NOOBS, making sure that SSH is enabled.
/ I hope you find this useful.

sudo apt-get update
sudo apt-get install apache2 php5 mysql-server libmysql-java ant unzip

[Enter password for mysql root user]

mysql -u root -p

// Change password for root access to mysql to 'myPassword'

mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('myPassword');

sudo /etc/init.d/mysql start

// sudo apt-get install openjdk-7-jdk

export JAVA_HOME=/usr/lib/jvm/jdk-8-oracle-arm32-vfp-hflt

echo "export JAVA_HOME=/usr/lib/jvm/jdk-8-oracle-arm32-vfp-hflt" >> ~/.bashrc

cd /tmp/
wget -c https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.5/bin/apache-tomcat-8.5.5.zip
unzip apache-tomcat-8.5.5.zip
sudo cp -a apache-tomcat-8.5.5 /usr/local/
export CATALINA_HOME=/usr/local/apache-tomcat-8.5.5
cd /usr/local
sudo ln -s $CATALINA_HOME tomcat
cd /usr/local/apache-tomcat-8.5.5/bin/
ls -l
chmod a+x *.sh
ls -l
$CATALINA_HOME/bin/startup.sh
echo "export CATALINA_HOME=/usr/local/apache-tomcat-8.5.5" >> ~/.bashrc
cd /tmp/
wget -c https://downloads.mysql.com/archives/get/file/mysql-connector-java-5.1.37.zip
unzip mysql-connector-java-5.1.37.zip
cd mysql-connector-java-5.1.37
ls -l
sudo cp mysql-connector-java-5.1.37-bin.jar $JAVA_HOME/jre/lib/ext
cd /tmp/
wget -c http://central.maven.org/maven2/javax/mail/javax.mail-api/1.5.2/javax.mail-api-1.5.2.jar
ls -l
sudo cp javax.mail-api-1.5.2.jar $JAVA_HOME/jre/lib/ext
sudo mv $JAVA_HOME/jre/lib/ext/javax.mail-api-1.5.2.jar $JAVA_HOME/jre/lib/ext/javax.mail.jar
wget -c https://sourceforge.net/projects/opengts/files/server-base/2.6.4/OpenGTS_2.6.4.zip/download
sudo mv download OpenGTS_2.6.4.zip
sudo unzip /tmp/OpenGTS_2.6.4.zip -d /usr/local
sudo chown -R pi:sudo /usr/local/OpenGTS_2.6.4
export GTS_HOME=/usr/local/OpenGTS_2.6.4
export ANT_HOME=/usr/share/ant
echo "export GTS_HOME=/usr/local/OpenGTS_2.6.4" >> ~/.bashrc
echo "export ANT_HOME=/usr/share/ant" >> ~/.bashrc
source ~/.bashrc
sudo ln -s $JAVA_HOME /usr/local/jvm
sudo ln -s $CATALINA_HOME /usr/local/tomcat
sudo ln -s $GTS_HOME /usr/local/gts
sudo ln -s /usr/lib/jvm/jdk-8-oracle-arm32-vfp-hflt /usr/local/java
nano $GTS_HOME/config.conf //uncomment db user and password lines.....
ls -l $CATALINA_HOME
unlink /usr/local/apache-tomcat-8.5.5/apache-tomcat-8.5.5
ls -l $CATALINA_HOME
cd $GTS_HOME
ant all
// Open mysql and enter: GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY 'myPassword' WITH GRANT OPTION;
bin/initdb.sh rootUser=root -rootPass=myPassword
bin/checkInstall.sh
bin/admin.sh Account -account=sysadmin -pass=myPassword -create
ant track
cp build/track.war $CATALINA_HOME/webapps
$CATALINA_HOME/bin/shutdown.sh
rm -rf $CATALINA_HOME/webapps/track
cp $GTS_HOME/build/track.war $CATALINA_HOME/webapps/
$CATALINA_HOME/bin/startup.sh
ant events
cp -v build/events.war $CATALINA_HOME/webapps/
ant gprmc
cp build/gprmc.war $CATALINA_HOME/webapps
bin/checkInstall.sh

Open browser at http://localhost:8080/track/Track and you are done.

//To change header, edit the file below as required then save....

Config.conf changes  Changes made
# -- private.xml: Page title
Domain.PageTitle=Cabs 4 U GPS Taxi Tracking
Domain.Copyright=All rights reserved Cabs4 U 2013
Domain.Properties.login.showPoweredByOpenGTS=false
Domain.Properties.login.showGTSVersion=false
Domain.Properties.login.hideBannerAfterLogin=true

OpenGts version 2.4.9
Private XML changes
    <!- Copyright used in html head 'meta' tag ->
    <Copyright>${Domain.Copyright=Copyright(C) 2013 Cabs 4 U All rights reserved}</Copyright>
    <!- Title used in html head 'title' tag ->
    <PageTitle i18n="PrivateXML.pageTitle">${Domain.PageTitle=${ServiceAccount.Name=Cabs 4 U} Online GPS Taxi Tracking}</PageTitle>

loginSession.jsp   - Hashed the entries below
    # <gts:var>
     # <td class="titleText" valign="center">
     #   ${pageTitle}<br>
      #  <font style="font-size: 8pt;"><i>(Powered by <a href="http://www.opengts.org" target="_blank" style="color:#444444;">OpenGTS</a>)</i></font>
     # </td>
      #</gts:var>
Changed
$GTS_HOME/src/org/opengts/db/LocalStrings_en.properties 
PrivateXML.pageTitle=Cabs 4 U GPS Taxi Tracking


/ Delete track folder and track.war from tomcat/webapps
/ Go to gts folder and do ant all
/ ant track.deploy
/ ant events.deploy
/ restart tomcat....
$CATALINA_HOME/bin/startup.sh

