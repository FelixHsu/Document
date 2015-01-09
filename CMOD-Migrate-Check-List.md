#Pre Step
	##確認並清除現有環境之垃圾(不存在的userid,AG,AP及OS file system中可確認不需要的資料)
	##Install new CMOD V9.0.0.3
		###new install AIX 7.1 				
			加語系ZH-TW,zh-tw,Zh-TW
			java jre 1.7 
			need add tmpvg 	--約10min
		###NEW CMOD server installation
			check environment
			lsvg -- rootvg and tmpvg
			lslpp -l | grep -i java  -- Java7.jre 
			lslpp -l | grep -i CDE -- GUI Environment
		###change file system size	
			chfs -a size=1G /
			chfs -a size=2G /tmp
			chfs -a size=2G /home
			chfs -a size=2G /var
			chfs -a size=5G /opt
		###Mount NIM:/TL/OD
			mkdir -p /mnt/OD
			mount NIM:/TL/OD /mnt/OD
		###install VNC , tar and unzip
			cd /mnt/OD/other/tightVNC
			rpm -ivh *.rpm
			cd /mnt/OD/other/tar
			rpm -ivh bash-4.3-12.aix5.1.ppc.rpm
			rpm -ivh gettext-0.10.40-8.aix5.2.ppc.rpm
			rpm -ivh libiconv-1.14-2.aix5.1.ppc.rpm
			rpm -ivh info-5.1-1.aix5.1.ppc.rpm
			rpm -ivh tar-1.27.1-1.aix5.1.ppc.rpm
			cd ..
			rpm -ivh unzip-6.0-2.aix5.1.ppc.rpm
		###start vnc server
			vncserver - set init password , confirm password, viewonly password (n)
			vncserver -kill :1
			cp xstartup /.vnc/
			vncserver
		###install db2 v10.1 fp3
			make file system /software
			cd /software
			/opt/freeware/bin/tar zxvf /mnt/OD/DB2/v10.1fp3_aix64_server.tar.gz
			/opt/freeware/bin/tar zxvf /mnt/OD/DB2/v10.1fp3_aix64_nlpack.tar.gz 
			cd server
			create filesystem /home/archive
				/usr/sbin/mklv -y'tvglv01' -t'jfs2' tempvg 10
				/usr/sbin/crfs -v jfs2 -d'tvglv01' -m'/home/archive' -A'yes'
				mount /home/archive
				chfs -asize=1G /home/archive
			create filesystem /usr/tivoli/tsm
				/usr/sbin/mklv -y'tvglv02' -t'jfs2' tempvg 10
				/usr/sbin/crfs -v jfs2 -d'tvglv02' -m'/usr/tivoli/tsm' -A'yes'
				mount /usr/tivoli/tsm
				chfs -asize=1G /usr/tivoli/tsm
			create group db2iadm1 -- GID 204 
				mkgroup -A id=204 db2iadm1 
			create user archive -- UID 205  pgrp db2iadm1
				mkuser -a id=205 admin='false' pgrp='db2iadm1' archive
				chown -R archive:db2iadm1 /home/archive
			launch vncviewer
			open cmd
			./db2setup
			/opt/IBM/db2/V10.1/cfg/db2ln
			ls -l /usr/lib | grep libdb2
			/opt/IBM/db2/V10.1/adm/db2licm -a /mnt/OD/db2/db2ese_o.lic
		###add root to db2iadm1
			chgrpmem -m + root db2iadm1
			lsgroup sysadm1 | awk '{print $4,$5}'
			echo ". /home/archive/sqllib/db2profile" >> /.profile; cat /.profile
		###install cmod 9 and 9.0.0.3
			cd /mnt/OD/9.0.0.0/CobolRTE/4.1 
			smitty install
			cd ../../
			smitty install  -- install gskit
			./odaix.bin -i console 
			cd ../9.0.0.3
			./odaix.bin -i console 
		###create ars used file system
			/usr/sbin/mklv -y'tvglv03' -t'jfs2' tempvg 10
			/usr/sbin/crfs -v jfs2 -d'tvglv03' -m'/ars/primarylog' -A'yes'
			mount /ars/primarylog
			chfs -asize=1G /ars/primarylog
			/usr/sbin/mklv -y'tvglv04' -t'jfs2' tempvg 10
			/usr/sbin/crfs -v jfs2 -d'tvglv04' -m'/ars/archivelog' -A'yes'
			mount /ars/archivelog 
			chfs -asize=1G /ars/archivelog
			/usr/sbin/mklv -y'tvglv05' -t'jfs2' tempvg 10
			/usr/sbin/crfs -v jfs2 -d'tvglv05' -m'/ars/cache1' -A'yes'
			mount /ars/cache1 
			chfs -asize=512M /ars/cache1
			/usr/sbin/mklv -y'tvglv06' -t'jfs2' tempvg 10
			/usr/sbin/crfs -v jfs2 -d'tvglv06' -m'/ars/acif' -A'yes'
			mount /ars/acif
			/usr/sbin/mklv -y'tvglv07' -t'jfs2' tempvg 10
			/usr/sbin/crfs -v jfs2 -d'tvglv07' -m'/ars/config' -A'yes'
			mount /ars/config
			/usr/sbin/mklv -y'tvglv08' -t'jfs2' tempvg 10
			/usr/sbin/crfs -v jfs2 -d'tvglv08' -m'/ars/tmp' -A'yes'
			mount /ars/tmp
		###set ars file system permission
			chown -R archive:db2iadm1 /ars/db1 /ars/primarylog /ars/archivelog /ars/tmp /ars/acif
			chmod 2770 /ars/db1
			chmod g+s /ars/db1
			chown -R root:system /ars/cache1
			chmod 700 /ars/cache1
			chmod g-s /ars/cache1
		###Edit ars config file
			vi /opt/IBM/ondemand/V9.0/config/ars.ini 
			vi /opt/IBM/ondemand/V9.0/config/ars.cfg
			vi /opt/IBM/ondemand/V9.0/config/ars.dbfs
			vi /opt/IBM/ondemand/V9.0/config/ars.cache
		###config CMOD
			export LANG=zh_TW
			/opt/IBM/ondemand/V9.0/bin/arsdb -I archive -gcv 
			/opt/IBM/ondemand/V9.0/bin/arssyscr -I archive -l
			/opt/IBM/ondemand/V9.0/bin/arssyscr -I archive -a 
			/opt/IBM/ondemand/V9.0/bin/arssyscr -I archive -m
		###grant archive db permission
			db2 connect to archive
			db2 "grant dbadm on database to user archive"
			db2 "grant bindadd on database to user archive"
			db2 "grant connect on database to user archive"
			db2 "grant createtab on database to user archive"
			db2 "grant create_external_routine on database to user archive"
			db2 "grant create_not_fenced_routine on database to user archive"
			db2 "grant implicit_schema on database to user archive"
			db2 "grant load on database to user archive"
			db2 "grant quiesce_connect on database to user archive"
			db2 "grant secadm on database to user archive"
			db2 "grant use of tablespace userspace1 to user archive"
			
Step 1. CMOD Prod full system flash copy	-- 一天之前通知FCB人員準備 所需時間 -- 約4-8hrs
		
Step 2. 修改設定 -- 約10分鐘
    hostname -- smitty hostname, vi /etc/hosts, uname -S <newhostname>, hostname <newhostname>
    安控設定、
    Crontab(root and archive)、
    inittab、
    TSM Client Hostname(建議使用Localhost) 
Step 3. 安裝前置作業 -- 約20分鐘
    建立檔案系統 -- /backup  /software
	修改檔案系統 -- /afpfont 從rootvg 改掛到 workvg
    修改檔案系統權限 -- chmod -R 777 /backup, chmod -R 777 /software
    掛載檔案系統 -- mkdir /od; mount NIM:/TL/OD /od
    解壓縮 perzl_org.tar 與ocm-webmin.tar到/soft -- cd /software; tar xf /mnt/TL/OD/perzl_org.tar; tar xf /mnt/TL/OD/ocm-webmin.tar;
    建立webmin and perl 連結 -- 
        ln -fs /software/ocm-webmin /opt/webmin; 
        mv /software/ocm-webmin/1.680/perl/bin/perl /software/ocm-webmin/1.680/perl/bin/perl-gcc4.8; 
        ln -fs /usr/bin/perl /opt/webmin/1.680/perl/bin/perl
    建立目錄 mkdir /opt/webmin/1.680/var
    啟動webmin -- /optwebmin/1.680/etc/start
	複製/rps2/RPS/rps 權限資料庫至 新伺服器或NIM:/TL/OD
Step 4. 開始昇級到CMOD 8.4.1.5 -- 約2小時
    開啟Browser, 連到webmin頁面
    執行 01-DB2 V6.1 To V7.2
            check ars, db2, perl, http, tsm service status
            if ars is running, kill it
            if db2 is running, stop it
            if tsm is running, stop it
            if jmgr perl is running, kill it.
            install db2 v7.2 - /od/DB2/db2v81/db2setup
            db2 migrate check -- db2ckmig
            migrate archive instance -- 
            Remove/create links to DB2 library files - /usr/lib/libdb2
            db2uext2 -- OnDemand DB2 log user exit link 
            Migrating ARCHIVE database(run as archive)
            Rebind packages(run as archive)
            Run statistics and reorganize OnDemand system tables(run as archive) -- arsdb -mv
            check and verify report is stable
        02-DB2 V7.2 to V8.1
            check ars, db2, perl, http, tsm service status
            if ars is running, kill it
            if db2 is running, stop it
            if tsm is running, stop it
            if jmgr perl is running, kill it.
            install tightVNC -- rpm -ivh /od/other/tightVNC/*.rpm
            set vncserver password -- vncpasswd 
            start vncserver
            use vncclient connect to vncserver
            install db2 v8.1 - /od/DB2/db2v81/db2setup
            run db2ckmig(run as archive) - /usr/opt/db2_08_01/bin/db2ckmig archive -l ~/db2ckmig.log -u archive -p archive
            modify root .profile and archive .profile
            re-login as root
            Migrating ARCHIVE instance(run as root) - /usr/opt/db2_08_01/instance/db2imigr -u db2fenc1 archive
            Remove/create links to DB2 files
            OnDemand DB2 log user exit library link - ln -fs /usr/lpp/ars/config/db2uext2.disk db2uext2
            Migrating ARCHIVE database(run as archive)
            Rebind packages(run as archive)
            Run statistics and reorganize OnDemand system tables(run as archive)
        03-CMOD V2.2.1.10 to V7.1.0.15
            check service status -- cmod, db2, tsm
            stop servics - cmod, stop db2, stop tsm
            backup CMOD config
            un-install 2.2.1
            check AIX C Set ++ runtime libraries
            install CMOD v7.1
            Upgrade CMOD 7.1.0.15
            Reconfigure OnDemand configuration files and scripts
            Upgrade OnDemand system tables
                Export the OnDemand system tables
                Drop the old tables
                Create the new tables and indexes
                Import the old table information from the exported files into the new tables
                run maintenance on the new tables
            Verify report is correct
        04-CMOD V7.1.0.15 to V7.1.2.15
            stop CMOD Server
            backup CMOD config
            install CMOD v7.1.1.0
            Upgrade CMOD 7.1.1.3
            Upgrade CMOD 7.1.2.0
            Upgrade CMOD 7.1.2.15
            Reconfigure OnDemand configuration files and scripts
            Upgrade OnDemand system - /usr/lpp/ars/bin/arsdb -uv
        05-TSM V3.7 to V5.5
            Stop CMOD/TSM server
            Backup TSM client option files
            Remove TSM 3.7 client
            Install tsm client 5.5
            Restore client option files
            Remove TSM 3.7 Server
            Install TSM 5.5 Server
            Start TSM
            Start CMOD
            Verify report is correct
        06-CMOD V7.1.2.15 to V8.4.1.5 -- 確認系統無誤，且可查到指定報表(含TSM)
            Stop the OnDemand Server. (run as root)
            Backup the OnDemand config
            Install the 8.4.0 (run as root)
            upgrade cmod db - /usr/lpp/ars/bin/arsdb -uv
            Install 8.4.0.3 Upgrade.(run as root)
            modify cmod config
            Start the OnDemand Server
            Stop the OnDemand Server. (run as root)
            Backup the OnDemand config
            Install the 8.4.1.0 Upgrade.(run as root)
            Update the database
                /usr/lpp/ars/bin/arsdb -gkv
                /usr/lpp/ars/bin/arsdb -I archive -vu
                /usr/lpp/ars/bin/arsdb -I archive -mv
                /usr/lpp/ars/bin/arssyscr -I archive -l
                su - root
                export LANG=zh_TW
                /usr/lpp/ars/bin/arstblsp -I archive -a 1 -g "System Log"
            Install the 8.4.1.5 Upgrade.(run as root)
            Verify report is correct
FCB 系統科備份
Step 5. 系統昇級到AIX7.1 --約需2小時
    FCB 系統人員進行AIX作業系統備份與昇級，		
        
FCB 系統科備份
Step 6. 系統全備份 -- 約需4 小時
    FCB 系統人員協助進行系統全備份 (all vg flash copy or rootvg and dbvg flash copy)，我們繼續安裝"應用程式伺服器"
Step 7. CMOD 昇級到 9.0.0.3 -- 約需120分鐘
    開啟Browser, 連到webmin頁面
    執行 08-DB2 V8.1 To V9.7 fp9
            check requirements (run as root)
            stop CMOD Server
            stop db2 service
            Precheck - bos.iocp.rte package needed
            remove libdb2.a link - /usr/opt/db2_08_01/cfg/db2rmln
            install DB2 V9.7  (run as root)
            Check DB2 cab be migrate - /opt/IBM/db2/V9.7/instance/db2ckupgrade archive -l /home/archive/db2ckupgrade-v97.log -u archive -p archive (run an archive)
            Upgrade Instance - /opt/IBM/db2/V9.7/instance/db2iupgrade -u db2fenc1 archive
            Recreate links to DB2 files - /opt/IBM/db2/V9.7/cfg/db2ln
            OnDemand DB2 log user exit - ln -fs /usr/lpp/ars/config/db2uext2.disk /home/archive/sqllib/adm/db2uext2
            Upgrading databases(run as archive) - db2 UPGRADE DATABASE archive USER archive USING archive
            Rebind packages(run as archive)
                db2 BIND /home/archive/sqllib/bnd/@db2ubind.lst BLOCKING ALL GRANT PUBLIC
                db2 BIND /home/archive/sqllib/bnd/@db2cli.lst BLOCKING ALL GRANT PUBLIC
            Run statistics and reorganize OnDemand system tables(run as archive) - /usr/lpp/ars/bin/arsdb -mv
            check indexes is type-2 indexes - db2IdentifyType1 -d archive -o convert-t1-indexes-dbname.db2
            indexrec database configuration parameter is set to RESTART - db2 get db cfg for ARCHIVE | grep INDEXREC
            Restart server and confirm report is correct
         09-CMOD V8.4.1.5 to V9.0.0.3
            Stop the OnDemand Server. (run as root)
            Backup the OnDemand database and config
            Install the 8.5.0.0 and 8.5.0.8 Upgrade.(run as root)
				install GSKit 8.0.13.4
				/mnt/OD/8.5.0.0/odaix.bin -i console
				/mnt/OD/8.5.0.5/odaix.bin -i console
            modify cmod config
            upgrade step(db2 need start)
				export LANG=zh_TW
                /usr/lpp/ars/bin/arsdb -I archive -efv
                /usr/lpp/ars/bin/arsdb -I archive -rv
                /usr/lpp/ars/bin/arsdb -I archive -mv
            Verify report is correct
            Stop the OnDemand Server. (run as root)
            Backup the OnDemand database and config
            Install the 9.0.0.0 and 9.0.0.3 Upgrade.(run as root)
				install Cobol rte 4.1
				install GSKit 8.0.14.21
				/mnt/OD/9.0.0.0/odaix.bin -i console
				/mnt/OD/9.0.0.3/odaix.bin -i console
            modify cmod config
            upgrade step(db2 need start)
				export LANG=zh_TW
                /opt/IBM/ondemand/V9.0/bin/arsdb -I archive -uv
                /opt/IBM/ondemand/V9.0/bin/arssyscr -I archive -l -u
                /opt/IBM/ondemand/V9.0/bin/arssyscr -I archive -a -u
                /opt/IBM/ondemand/V9.0/bin/arsdb -I archive -efv
                /opt/IBM/ondemand/V9.0/bin/arsdb -I archive -rv
                /opt/IBM/ondemand/V9.0/bin/arsdb -I archive -mv
            Verify report is correct
         10-DB2 V9.7 fp9 to db2 v10.1 fp3
            take your cmod server offline
            take your DB2 server offline
            remove libdb2.a link
            install DB2 V10.1  (run as root)
            Check DB2 can be migrate -- db2 need start, run as archive
				/opt/IBM/db2/V10.1/instance/db2ckupgrade archive -l /home/archive/db2ckupgrade-v10.1.log 
            Upgrade Instance -- db2 need stop, run as root
				/opt/IBM/db2/V10.1/instance/db2iupgrade -u db2fenc1 archive
            Updating the DB2 license key(run as root)
				/opt/IBM/db2/V10.1/adm/db2licm -a /mnt/OD/db2/db2ese_o.lic
            OnDemand DB2 log user exit - ln -fs /opt/IBM/ondemand/V9.0/config/db2uext2.disk /home/archive/sqllib/adm/db2uext2
            Recreate links to DB2 library file
				/opt/IBM/db2/V10.1/cfg/db2ln 
            Upgrading databases -- /ars/primarylog must had 5GB, run as archive
				chfs -asize=5G /ars/primarylog
				db2start
				db2 connect to arrchive
				db2 "upgrade database archive rebindall"
            Run statistics and reorganize OnDemand system tables
                /opt/IBM/ondemand/V9.0/bin/arsdb -gkv
                /opt/IBM/ondemand/V9.0/bin/arsdb -mv
            restart AIX
            Verify report is correct

Step 8. 	change afpfont of rootvg to workvg
			cp PERL 5.8.6 module to workvg
			cp /rps2/RPS/rps to workvg
			cp /etc/hosts.lpd to workvg
			
Step 9. new server takeover 約需120分鐘(依webmin執行)
            10.10.74.16
			varyoffvg workvg
			exportvg workvg
			varyoffvg cachevg
			exportvg cachevg
			varyoffvg dbvg
			exportvg dbvg
			
			10.10.74.11
			varyoffvg tempvg
			exportvg tempvg
			importvg -y dbvg hdisk1(00f6b99f1d3d32ed) hdisk2(00f6b99f1d3d339a) hdisk3(00f6b99f1d3d342a)
			importvg -y  workvg hdisk4(00f6b99f0f601875)
			importvg -y  cachevg hdisk5(00f6b99f0f5fb36c)
        執行CMOD報表日期錯誤資料修正
			change /opt/webmin/1.680/perl/bin/perl link
				unlink /opt/webmin/1.680/perl/bin/perl
				ln -fs /opt/webmin/1.680/perl/bin/perl5.18.2 /opt/webmin/1.680/perl/bin/perl
			install gcc ( for perl 5.18.2 use)
			set archive password 
			1.datacorrect.pl
				/mnt/OD/datacorrect/bin/datacorrect.pl 
			2.ID_clear.pl
				/mnt/OD/cmod-admin-hang/ID_clear.pl archive password root.arsagperms
				/mnt/OD/cmod-admin-hang/ID_clear.pl archive password root.arsagperms
Step 10. OCM安裝 -- 約需60分鐘
	DB2 tcp/ip Config 
		cat /etc/services | grep -i db2
		db2 get dbm cfg | grep -i svcename | grep -v SSL
		db2 update dbm cfg SVCENAME DB2_archive (same as /etc/services config)
		db2set -all
		db2set DB2COMM=tcpip
	chech unzip
		rpm -qa | grep -i unzip
	set java version
	create ocmadmin (password set as password)
		mkgroup ocmadmin
		mkuser admin=false pgrp=ocmadmin ocmadmin
		passwd ocmadmin
	Enable IBM DB2 Federation feature ( db2 must restart)
		db2 get dbm cfg | grep -i FEDERATED
		db2 update dbm cfg using FEDERATED yes
		db2 get dbm cfg | grep -i FEDERATED
		/etc/rc.d/init.d/cmod stop
		/etc/rc.d/init.d/cmod start
	Create file system for ocm server install
		/usr/sbin/mklv -y'wvglv11' -t'jfs2' workvg 1
		/usr/sbin/crfs -v jfs2 -d'wvglv11' -m'/opt/steam' -A'yes'
		mount /opt/steam
		chfs -asize=2G /opt/steam
		/usr/sbin/mklv -y'wvglv12' -t'jfs2' workvg 1
		/usr/sbin/crfs -v jfs2 -d'wvglv12' -m'/opt/steam/ocm' -A'yes'
		mount /opt/steam/ocm
		chfs -asize=512M /opt/steam/ocm
		/usr/sbin/mklv -y'wvglv13' -t'jfs2' workvg 1
		/usr/sbin/crfs -v jfs2 -d'wvglv13' -m'/opt/steam/data' -A'yes'
		mount /opt/steam/data
		chfs -asize=512M /opt/steam/data
		/usr/sbin/mklv -y'wvglv14' -t'jfs2' workvg 1
		/usr/sbin/crfs -v jfs2 -d'wvglv14' -m'/opt/steam/logs' -A'yes'
		mount /opt/steam/logs
		chfs -asize=512M /opt/steam/logs
		chown -R ocmadmin:ocmadmin /opt/steam /opt/steam/ocm /opt/steam/data /opt/steam/logs
	check time zone
		cat /etc/environment | grep -i tz
		TZ=Asia/Taipei
	install ocm server
		archive password --> password
		ocmadmin password --> password
Step 11. others -- 約需30分鐘
    LPD config
		mkdir -p /opt/steam/data/watcher/rptlst
        smit lpd -> Start the Print Server Subsystem (lpd daemon)
            Start subsystem now, on system restart, or both --> both
        smit lpd -> Add Print Access for a Remote Client
            127.0.0.1
            localhost
        smit mkpq
            Attachment Type -> other (User Defined Backend)
            Name of QUEUE to add -> P2F
            Name of QUEUE DEVICE to add -> p2fdev
            BACKEND PROGRAM pathname -> /report/bin/mvsp2f /opt/steam/data/watcher/rptlst /report/backup
        check queue status    
			enq -e
        Create /report file system
            /usr/sbin/mklv -y'wvglv21' -t'jfs2' rootvg 80
            /usr/sbin/crfs -v jfs2 -d'wvglv21' -m'/report' -A'yes'
            mount /report
            chfs -asize=1G /report
            mkdir /report/test /report/backup /report/mvslog /report/bin
        Install backend program
            cp -p /mnt/OD/mvcp2f/bin/* /report/bin
            cd /report/bin
            chmod +x mvsp2f
            chmod +x getqinfo
        Change folder owner
            chown -R lpd.printq /report
            chown -R lpd.printq /opt/steam/data/watcher/rptlst
        Localtest
            cd /mnt/OD/mvcp2f/t
			./run_aix.sh
    sftp config
        vi /etc/ssh/sshd_config
            SyslogFacility LOCAL7
            LogLevel VERBOSE
            Subsystem       sftp    /usr/sbin/sftp-server -f AUTH -l VERBOSE
        vi /etc/syslog.conf
            local7.info     /var/log/ssh/sshd.log rotate time 1d
            auth.info       /var/log/ssh/sftp-ai.log rotate time 1d
        mkdir /var/log/ssh
        touch /var/log/ssh/sftp-ai.log
        touch /var/log/ssh/sshd.log
        refresh -s syslogd (or  stopsrc -s syslogd and startsrc -s syslogd)
        stopsrc -g ssh (or stopsrc -s sshd)
        startsrc -g ssh(or startsrc -s sshd)

Step 12. verify -- 約需30分鐘
		set System log rotate job
		set Ap log rotate job
step 13. AG Recreate, Cache reconfig

Post Step
	1.rename lv name(smit chlv，統一命名)
	2.tar & unmount file systems 	(rps, rps2, sysmgr, usr/opt/jboss-4.0.3SP1
									jmr-old, reptmgmt-old, odmigr, reptmgmt, jmr
									usr/opt/apache-tomcat-6.0.35, worktmp)
	3.依手冊執行系統強化相關設定(待felix安排時間來測並評估時間)


orig file system INFO
[root@ondemand2]/ # lsvg
rootvg
dbvg
cachevg
workvg

[root@ondemand2]/ # lsvg -l rootvg
rootvg:
LV NAME             TYPE       LPs     PPs     PVs  LV STATE      MOUNT POINT
hd5                 boot       1       1       1    closed/syncd  N/A
hd6                 paging     96      96      1    open/syncd    N/A
hd8                 jfslog     1       1       1    open/syncd    N/A
hd4                 jfs        31      31      1    open/syncd    /
hd2                 jfs        80      80      1    open/syncd    /usr
hd9var              jfs        64      64      1    open/syncd    /var
hd3                 jfs        11      11      1    open/syncd    /tmp
hd1                 jfs        40      40      1    open/syncd    /home
hd10opt             jfs        68      68      1    open/syncd    /opt
lv08                jfs        4       4       1    open/syncd    /afpfont
lv03                jfs2       24      24      1    open/syncd    /rps2
lg_dumplv           sysdump    16      16      1    open/syncd    N/A
lv05                jfs2       16      16      1    open/syncd    /usr/opt/jboss-4.0.3SP1
lv02                jfs2       16      16      1    open/syncd    /sysmgr
lv39                jfs2log    1       1       1    open/syncd    N/A
lv12                jfs2       4       4       1    open/syncd    /jmr-old
lv24                jfs2       4       4       1    open/syncd    /reptmgmt-old
lv10                jfs2       2       2       1    open/syncd    /usr/opt/apache-tomcat-6.0.35
[root@ondemand2]/ # lsvg -l dbvg
dbvg:
LV NAME             TYPE       LPs     PPs     PVs  LV STATE      MOUNT POINT
fslv43              jfs2log    1       1       1    open/syncd    N/A
fslv44              jfs2       220     220     3    open/syncd    /ars/db1
fslv45              jfs2       30      30      3    open/syncd    /ars/db2
[root@ondemand2]/ # lsvg -l cachevg
cachevg:
LV NAME             TYPE       LPs     PPs     PVs  LV STATE      MOUNT POINT
fslv19              jfs2log    1       1       1    open/syncd    N/A
lv00                jfs2       210     210     1    open/syncd    /od_history
fslv01              jfs2       8       8       1    open/syncd    /dsmdump
fslv00              jfs2       9       9       1    open/syncd    /adsmbkup
fslv20              jfs2       4       4       1    open/syncd    /rps
fslv42              jfs2       4       4       1    open/syncd    /odmigr
fslv18              jfs2       1       1       1    open/syncd    /ars/tmp
fslv06              jfs2       4       4       1    open/syncd    /ars/acif
fslv22              jfs2       3       3       1    open/syncd    /ars/primarylog
fslv23              jfs2       1       1       1    open/syncd    /ars/archivelog
fslv21              jfs2       90      90      1    open/syncd    /ars/cache2
fslv07              jfs2       90      90      1    open/syncd    /ars/cache3
fslv10              jfs2       90      90      1    open/syncd    /ars/cache4
fslv11              jfs2       90      90      1    open/syncd    /ars/cache5
fslv09              jfs2       90      90      1    open/syncd    /ars/cache6
fslv13              jfs2       90      90      1    open/syncd    /ars/cache7
fslv14              jfs2       90      90      1    open/syncd    /ars/cache8
fslv15              jfs2       90      90      1    open/syncd    /ars/cache9
fslv16              jfs2       90      90      1    open/syncd    /ars/cache10
fslv25              jfs2       90      90      1    open/syncd    /ars/cache11
fslv26              jfs2       90      90      1    open/syncd    /ars/cache12
fslv27              jfs2       90      90      1    open/syncd    /ars/cache13
fslv28              jfs2       90      90      1    open/syncd    /ars/cache14
fslv29              jfs2       90      90      1    open/syncd    /ars/cache15
fslv30              jfs2       90      90      1    open/syncd    /ars/cache16
fslv31              jfs2       90      90      1    open/syncd    /ars/cache17
fslv17              jfs2       90      90      1    open/syncd    /ars/cache1
[root@ondemand2]/ # lsvg -l workvg
workvg:
LV NAME             TYPE       LPs     PPs     PVs  LV STATE      MOUNT POINT
lv01                jfs2log    1       1       1    open/syncd    N/A
lv06                jfs2       160     160     1    open/syncd    /report
lv07                jfs2       18      18      1    open/syncd    /reptmgmt
lv09                jfs2       340     340     1    open/syncd    /jmr
lv04                jfs2       4       4       1    open/syncd    /worktmp
lv11                jfs2       12      12      1    open/syncd    /audit
bkuplv01            jfs2       10      10      1    closed/syncd  /backup
softlv01            jfs2       10      10      1    closed/syncd  /software
[root@ondemand2]/ # 

new file system ref
				Filesystem    GB blocks      Free %Used    Iused %Iused Mounted on
system			/dev/hd4           2.00      1.45   28%    18174     6% /
system			/dev/hd2           4.88      1.77   64%    53463    12% /usr
改lv			/dev/hd5           1.00      0.66   35%     6707     4% /var
system			/dev/hd3           1.19      0.74   38%      368     1% /tmp
system			/dev/hd1           1.00      0.99    1%       78     1% /home
system			/proc                 -         -    -         -     -  /proc
改lv			/dev/hd6           5.44      0.76   87%    36050    16% /opt
mig用			/dev/livedump      0.25      0.25    1%        4     1% /var/adm/ras/livedump
mig用			/dev/softlv01      5.00      1.58   69%    32531     9% /soft
改lv			/dev/arslv1      220.00     33.97   85%     3598     1% /ars/db1
改lv			/dev/arslv2       30.00     21.25   30%      618     1% /ars/db2
mig用			/dev/dbvglv10      6.00      3.37   44%     1555     1% /home/archive
dbvg改用workvg	/dev/lv06         80.00     60.24   25%    50372     1% /report
cachevg			/dev/lv00        210.00      9.20   96%      213     1% /od_history
cachevg			/dev/fslv01        8.00      1.16   86%     1756     1% /dsmdump
cachevg			/dev/fslv00        9.00      1.34   86%       16     1% /adsmbkup
remove			/dev/fslv20        4.00      0.43   90%    47629    31% /rps
remove			/dev/fslv42        4.00      1.45   64%       62     1% /odmigr
cachevg			/dev/fslv18        1.00      1.00    1%      573     1% /ars/tmp
cachevg			/dev/fslv06        4.00      4.00    1%        4     1% /ars/acif
cachevg			/dev/fslv22        5.00      1.15   77%      261     1% /ars/primarylog
cachevg			/dev/fslv23        3.00      3.00    1%        4     1% /ars/archivelog
cachevg			/dev/fslv21       90.00      9.42   90%   825771    28% /ars/cache2
cachevg			/dev/fslv07       90.00      9.42   90%   810866    27% /ars/cache3
cachevg			/dev/fslv10       90.00      9.42   90%   822818    28% /ars/cache4
cachevg			/dev/fslv11       90.00      9.42   90%   824266    28% /ars/cache5
cachevg			/dev/fslv09       90.00      9.42   90%   806794    27% /ars/cache6
cachevg			/dev/fslv13       90.00      9.42   90%   818717    28% /ars/cache7
cachevg			/dev/fslv14       90.00      9.42   90%   820901    28% /ars/cache8
cachevg			/dev/fslv15       90.00      9.42   90%   813958    28% /ars/cache9
cachevg			/dev/fslv16       90.00      9.42   90%   816768    28% /ars/cache10
cachevg			/dev/fslv25       90.00      9.41   90%   856279    28% /ars/cache11
cachevg			/dev/fslv26       90.00      9.42   90%   845431    28% /ars/cache12
cachevg			/dev/fslv27       90.00      9.42   90%   849698    28% /ars/cache13
cachevg			/dev/fslv28       90.00      9.42   90%   800435    27% /ars/cache14
cachevg			/dev/fslv29       90.00      9.42   90%   806981    27% /ars/cache15
cachevg			/dev/fslv30       90.00      9.42   90%   838207    28% /ars/cache16
cachevg			/dev/fslv31       90.00      9.42   90%   814601    28% /ars/cache17
cachevg			/dev/fslv17       90.00      9.42   90% 17893573    90% /ars/cache1
mig				/dev/cachevglv82      1.00      1.00    1%       16     1% /opt/IBM/ondemand/V9.0/config
mig				/dev/cachevglv83      8.00      5.84   27%      325     1% /usr/tivoli/tsm
cachevg			/dev/cachevglv18     30.00     29.92    1%       86     1% /ars/cache18
cachevg			/dev/cachevglv19     30.00     29.92    1%      380     1% /ars/cache19
cachevg			/dev/cachevglv20     30.00     29.92    1%       79     1% /ars/cache20
workvg			/dev/lv07          9.00      8.76    3%     3750     1% /reptmgmt
workvg			/dev/lv09        170.00     62.47   64%   208584     2% /jmr
workvg			/dev/lv04          2.00      1.65   18%     1498     1% /worktmp
workvg			/dev/lv11          6.00      3.95   35%        9     1% /audit
workvg			/dev/bkuplv01      5.00      1.21   76%       70     1% /backup
mig				/dev/fslv02        5.00      1.43   72%    32683     9% /software
workvg			/dev/ocmlv01       1.00      0.85   16%      401     1% /opt/steam/ocm
workvg			/dev/ocmlv02      11.00      9.82   11%      523     1% /opt/steam/data
workvg			/dev/ocmlv03       1.00      0.95    5%       35     1% /opt/steam/logs
				/aha                  -         -    -        18     1% /aha
				NIM:/TL/OD        30.00      4.33   86%    28109     3% /mnt/OD

				
rootvg 		rvglv
dbvg		dvglv
cachevg		cvglv
workvg		wvglv

db2 update dbm cfg using diagsize 1024

10.10.74.16 
lspv
hdisk0          00f6b99fbf8ef2ec                    rootvg          active
hdisk6          00f6b99fc40d9729                    None
hdisk1          00f6b99f1d3d32ed                    dbvg            active
hdisk2          00f6b99f1d3d339a                    dbvg            active
hdisk3          00f6b99f1d3d342a                    dbvg            active
hdisk4          00f6b99f0f601875                    workvg          active
hdisk5          00f6b99f0f5fb36c                    cachevg         active

10.10.74.11 
lspv
hdisk5          00f6b99f0f5d21c9                    rootvg          active
hdisk6          00f6b99f1d3d32ed                    None
hdisk7          00f6b99f1d3d339a                    None
hdisk8          00f6b99f1d3d342a                    None
hdisk9          00f6b99f0f601875                    None
hdisk10         00f6b99f0f5fb36c                    None
