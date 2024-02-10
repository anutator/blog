---
title: bash скрипты и файлы CMS для изучения
share: "true"
---
Скрипты и файлы для изучения

Файл */etc/rc2.d/S98cms_ndd*. В случае, если на CMS два порта eth, нужно запретить маршрутизацию пакетов с одного порта на другой. У нас,во-первых, только один eth0, а во-вторых, опция включена.

``` title="/etc/rc2.d/S98cms_ndd" hl_lines="1"
ndd -set /dev/ip ip_forwarding 0
ndd -set /dev/ip ip_strict_dst_multihoming 1
ndd -set /dev/ip ip_forward_directed_broadcasts 0
ndd -set /dev/ip ip_forward_src_routed 0
ndd -set /dev/ip ip_respond_to_echo_broadcast 0
ndd -set /dev/ip ip_respond_to_timestamp_broadcast 0
ndd -set /dev/ip ip_respond_to_address_mask_broadcast 0
ndd -set /dev/ip ip_ignore_redirect 1
ndd -set /dev/ip ip_send_redirects 0
ndd -set /dev/ip ip_respond_to_timestamp 0
ndd -set /dev/tcp tcp_conn_req_max_q0 2048
ndd -set /dev/tcp tcp_conn_req_max_q 1024
```

Содержимое каталога */opt/informix/bin* (почти всё — двоичные файлы)

```bash
(mrc-krl15-ucacms1)-(root)=# ls /opt/informix/bin
about_files    esqlvers             infoshp          onedpu         onspaces
about.html     filtersym.sh         infxmsg          oninit         onsrvapd
archecker      finderr              instterm         onload         onstat
blademgr       GenMacKey            ipload           onlog          ontape
cdr            genoncfg             libcairo-swt.so  onmode         onunload
check_version  glfiles              loadshp          onparams       plugins
chkenv         hdrmkpri.sh          msgfile          onpassword     rofferr
configuration  hdrmksec.sh          onaudit          onperf         setenv
crtcmap        ibmifmx_security.sh  onbar            onpladm        setisqlenv
db2dbgm.jar    idsd                 onbar_d          onpload        snmpdm
dbaccess       ifmxgcore            oncheck          onpsm          txbsapswd
dbexport       ifxbkpcloud.jar      onclean          onrestorept    unloadshp
dbimport       ifxclone             oncmsm           onsecurity     xtrace
dbinit.sh      ifxcollect           onconfig_diff    onshowaudit    xtree
dbload         ifxdeploy            ondblog          onshutdown.sh
dbschema       ifxdeployassist      ondwachk         onsmsync
drdaprint      ifxpipecat           onedcu           onsnmp
```

Содержимое каталога */cms/toolsbin*

```bash
(mrc-krl15-ucacms1)-(root)=# ls /cms/toolsbin
age_pw                   chktunes   db_util    lib        parsecv         setSimLink  tsdatemod
age_pw_exclude_template  clint      diff_rtdb  link_perf  perfcmd.sh      showtime    ts_mod
age_pw_template          cmsu       fakepc     maintlock  PerfHistLogger  slem        ttlist
awk_rtdb                 cow_paste  getlog     mqtool     rebuild_dbtemp  swversion   tz_paste
chk_ext                  custfiles  GET.METS   netconfig  reset_tenants   tenRptMig
```

### Скрипт /opt/Informix/bin/hdrnksec.sh

```bash title="/opt/informix/bin/hdrmksec.sh"
#!/bin/sh
#  Title      : hdrmksec.sh
#  Description: This script changes the type of the Data replication server
#               to Secondary. This script will soon be depricated. Need to
#               use command line steps after that.

cat << EOF

This script changes the type of the Data replication server to Secondary.

Steps to switch server types in an HDR pair:

  Instance A (currently Primary)          Instance B (currently Secondary)
  ------------------------------          --------------------------------
  1] onmode -ky                              (server should be up)
                                          2] hdrmkpri.sh <primary_server_name>
  3] hdrmksec.sh <secondary_server_name>
     (now a Secondary server)             4] oninit (now a Primary server)
EOF

# Check usage
if (test $# -gt 1) then
    echo " "
    echo "Usage: $0 [paired server name]"
    echo " "
    exit 1
fi

cat << EOF

WARNING: Please ensure the following before proceeding further:

           1] the paired  database server is OFFLINE, and
           2] the current database server has Data replication turned off.

         Not doing so, shall make the two database servers in the
         Data replication pair out-of-sync, and shall require
         re-establishing the pair.

Press return to continue...
EOF
read ignore_var

server_up=`onstat - | egrep -- "-- Up"`
if (test "${server_up}") then
    onstat_g_dri=`onstat -g dri | egrep -i "standard|primary|secondary"`
    dri_type=`echo ${onstat_g_dri}          | awk '{ print $1 }'`
    dri_state=`echo ${onstat_g_dri}         | awk '{ print $2 }'`
    dri_paired_server=`echo ${onstat_g_dri} | awk '{ print $3 }'`
if (test "${dri_type}" = "HDR" || test "${dri_type}" = "SDS" || test "${dri_type}" = "RSS") then
        dri_type=`echo ${onstat_g_dri}         | awk '{ print $2 }'`
        dri_state=`echo ${onstat_g_dri}        | awk '{ print $3 }'`
        dri_paired_server=`echo ${onstat_g_dri} | awk '{ print $4 }'`
fi

    echo " "
    echo " "
    echo "Replication Status: Type          : ${dri_type}"
    echo "                    State         : ${dri_state}"
    echo "                    Paired server : ${dri_paired_server}"

    if (test "${dri_type}" = "secondary") then
        echo " "
        echo "ERROR: Database server is already a Secondary server."
        echo "This script should be run on a Primary or Standard server only."
        echo " "
        exit 1
    fi

    if (test "${dri_state}" = "on") then
        echo " "
        echo "ERROR: Please ensure that the paired database server in the"
        echo "       Data replication pair is OFFLINE and data replication"
        echo "       is turned OFF, before proceeding further."
        echo " "
        exit 1
    fi

    echo " "
    echo "Shutting down the current database server via:"
    echo "      onmode -ky"
    onmode -ky
else
    echo " "
    echo "Current database server is down/off-line."
fi # End of if (test "${server_up}"...


echo " "
echo "Run \$INFORMIXDIR/bin/hdrmkpri.sh on the paired server."
echo "After the run OR if it has already been run, then"
echo "Press Return to continue ..."
read ignore_var

# Note: The paired server name is available only for Primary/Secondary type,
# and NOT for Standard type.
#
paired_server=""
if (test $# -eq 0) then
    if (test "${dri_paired_server}") then
        paired_server=${dri_paired_server}
    fi
else
    paired_server=$1
fi

if (test ! "${paired_server}") then
    echo " "
    echo "Enter the paired server name and press return:"
    read paired_server
fi

# A change from Primary to Standard type is NOT needed.

echo " "
# Do Physical recovery and bypass logical recovery from disk.
echo "Starting the database server via :"
echo "  oninit -PHY"
echo "  Note : -PHY is an undocumented option only used in this"
echo "         HDR failover script"
oninit -PHY

sleep 2

echo " "
echo "Changing to Secondary type via: (could take a couple of minutes)"
echo "  onmode -d secondary ${paired_server}"

# This is needed, as 'oninit -PHY' resets HDR type and state information,
# and new HDR type and state information changed via 'onmode -d secondary'
# do not make it to the disk, unless the HDR pair becomes operational.
# This is because the HDR state and type changes are stored in the reserved
# pages, which are updated via checkpoints and checkpoints can happen on a
# Secondary database server only through the Primary database server.
#
echo " "
echo "WARNING: The current database server should NOT be shutdown now."
echo "         If it is shutdown for any reason, then you need to re-run"
echo "         this script again from the start."

echo " "
echo "Start the paired (now Primary) database server, while the current"
echo "(now Secondary) database server tries to connect to it, to make the"
echo "Data replication pair operational."
echo " "
```
### Расширенная информация о системе
swversion (*/cms/toolsbin/swversion*)

```bash title="/cms/toolsbin/swversion"
#!/bin/ksh
# NAME: swversion
# DESCRIPTION:  Shell to display CMS version, platform and OS information
# Arguments:  none
# Installed: /cms/toolsbin/swversion
# Usage: swversion
#
# MODIFICATIONS:
# DATE        WHO   DESCRIPTION
# 08/25/17    sms   CMS-1731  Inital development
# 01/08/18    dsb   CMS-1830  Change the name of cms_info to swversion

uname -a | grep -i linux > /dev/null
LINUX=$?
uname -a | grep -i sparc > /dev/null
SPARC=$?
uname -a | grep -i sunos > /dev/null
SUNOS=$?
uname -a | grep -i i86pc > /dev/null
SUNx86=$?
VMWARE=1
AMAZON=1
if [ $LINUX -eq 0 ];
then
        echo CMS is running on Linux
        dmidecode | grep -i vmware > /dev/null
        VMWARE=$?
        dmidecode | grep -i amazon > /dev/null
        AMAZON=$?
elif [ $SUNOS -eq 0 ]
then
        if [ $SPARC -eq 0 ]
        then
                echo CMS is running on Solaris SPARC
        elif [ $SUNx86 -eq 0 ]
        then
                echo CMS is running on Solaris x86
        fi
fi
if [ $VMWARE -eq 0 ];
then
        echo CMS is running on VMware
elif [ $AMAZON -eq 0 ]
then
        echo CMS is running on Amazon AWS
else
        echo CMS is running on physical hardware
fi

if [ $LINUX -eq 0 ];
then
        rpm -qi cms | grep "Summary"
        echo -n "Version     : "
        rpm -qa | grep -i "^cms-"
        rpm -qi cmsweb | grep "Summary"
        echo -n "Version     : "
        rpm -qa | grep -i "^cmsweb-"
elif [ $SUNOS -eq 0 ]
then
        pkginfo -l cms | grep "NAME"
        pkginfo -l cms | grep "VERSION"
        pkginfo -l cmsweb | grep "NAME"
        pkginfo -l cmsweb | grep "VERSION"
fi
```

Пример вывода:

```bash
# /cms/toolsbin/swversion
CMS is running on Linux
CMS is running on VMware
Summary     : Avaya(TM) Call Management System R18.0.2.0
Version     : cms-R18.0.2.0-ma.k.x86_64
Summary     : Avaya(TM) CMS Web Supervisor
Version     : cmsweb-R18.0.2.0-ma.k.x86_64
```

### Оболочки
Какие оболочки shell есть в Linux (Avaya CMS). Вариант 2 — использовать команду смены оболочки `chsh` (`change shell`) с параметром `-l` (` --list-shells`) :

```bash
# cat /etc/shells
/bin/sh
/bin/bash
/sbin/nologin
/bin/dash
/bin/ksh
/bin/tcsh
/bin/csh

# chsh -l
/bin/sh
/bin/bash
/sbin/nologin
/bin/dash
/bin/ksh
/bin/tcsh
/bin/csh
```

### Пароли
*/cms/toolsbin/age_pw*

```bash title="/cms/toolsbin/age_pw"
#!/bin/ksh
# Script name: age_pw (LINUX Version)
# This program invokes password aging for all cms users. New users created
# will also have their passwords aged.
# To match age_pw.sh on the Sun, the user will be prompted for the password
# aging interval in weeks. It will be stored in days in PWAGE_FILE.
#
# MODIFICATIONS
# DATE     WHO           DESCRIPTION
#--------------------------------------------------------------------------
# 07/12    ctb           wi01033398: Initial Linux version.
# 10/12    ctb           wi01049926: Linux: passwd command gets error on perms
#                                    for /etc/login.defs
# **************************************************************************
#                                 age_pw
# **************************************************************************

OSNAME=`uname`
# Security precautions
#
if [ "$OSNAME" = "SunOS" ]; then
PATH=/usr/bin:/usr/sbin:/usr/5bin; export PATH
fi
IFS="
"; export IFS
DEFAULTIFS=$IFS
trap ":" 0 2 3 4 5 6 7 8 10 12 13 14 15
#
# End security precautions

# **************************************************************************
# Function:     check_for_quit()
# Description:  Verifies if the user types "q" to a menu prompt.  The
#               function examines the content's of the global ksh built-in
#               variable REPLY and take the appropriate action.
# Arguments:    None.
# Return Value: None.
# **************************************************************************
function check_for_quit
{
  if [ "$REPLY" = "q" ]
  then
    cleanup_exit 0
  else
    display "Invalid selection!\n"
  fi
} # End check_for_quit()

# **************************************************************************
# Function:     check_for_yorn()
# Description:  Verifies if the user types "y", "n" or "q" to a menu prompt.
#               The function requires that string PS3 has already been set
#               by the caller function.  The default is set to yes (y).
# Arguments:    $1 = name of variable which the valid response (y|n) is
#                    placed in.  The resulting value placed in $1 is always
#                    "y" or "n".
# Return Value: None.
# Note:         Here's an example showing how to use this function:
#               check_for_yorn RESPONSE
#               if [ "$RESPONSE" = "y" ]; then
#                 print "the answer is yes"
#               else
#                 print "the answer is no"
#               fi
# **************************************************************************
function check_for_yorn
{
  typeset L_VARNAME=$1

  while true
  do
    display -n "$PS3 (default y) "
    read REPLY
    test -z "$REPLY" && REPLY=y
    case "$REPLY" in
      n|y) break;;
      *  ) check_for_quit;;
    esac
  done
  eval "${L_VARNAME}=\"${REPLY}\""
} # End check_for_yorn()

# **************************************************************************
# Function:     check_for_123()
# Description:  The function requires that string PS3 has already been set
#               by the caller function.  The default is set to 1.
# Arguments:    $1 = name of variable which the valid response (y|n) is
#                    placed in.  The resulting value placed in $1 is always
#                    "y" or "n".
# Return Value: None.
# Note:         Here's an example showing how to use this function:
#               check_for_123 RESPONSE
#               case "$RESPONSE" in
#                  1) print "You chose 1" ;;
#                  2) print "You chose 2" ;;
#                  3) print "You chose 3" ;;
#               esac
# **************************************************************************

function check_for_123
{
  typeset L_VARNAME=$1

  while true
  do
    display -n "$PS3 (default 1) "
    read REPLY
    test -z "$REPLY" && REPLY=1
    case "$REPLY" in
      1|2|3) break;;
      *  ) check_for_quit;;
    esac
  done
  eval "${L_VARNAME}=\"${REPLY}\""
} # End check_for_123()

# **************************************************************************
# Function:     check_range()
# Description:  The function requires that string PS3 has already been set
#               by the caller function.  The default is set to 1.
# Arguments:    $1 = name of variable which the valid response (y|n) is
#                    placed in.  The resulting value placed in $1 is always
#                    "y" or "n".
# Return Value: None.
# Note:         Here's an example showing how to use this function:
#               check_range RESPONSE
#               case "$RESPONSE" in
#                  1) print "You chose 1" ;;
#                  2) print "You chose 2" ;;
#                  3) print "You chose 3" ;;
#                    ...
#                  52) print "You chose 52" ;;
#               esac
# **************************************************************************
function check_range
{
  typeset L_VARNAME=$1
  while true
  do
    display -n "$PS3 (default $DEF_WEEKS) "
    read REPLY
    test -z "$REPLY" && REPLY=$DEF_WEEKS
    case "$REPLY" in
      1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|16|17|18|19|20|\
     21|22|23|24|25|26|27|28|29|30|31|32|33|34|35|36|37|38|39|40| \
     41|42|43|44|45|46|47|48|49|50|51|52) break;;
      *  ) check_for_quit;;
    esac
  done
  eval "${L_VARNAME}=\"${REPLY}\""
} # End check_range()

# **************************************************************************
# Function:     display()
# Description:  Displays messages to screen
# Arguments:    $1 = message to be displayed.
# Return Value: None.
# **************************************************************************
function display
{
  print "$@"
} # End display()

# **************************************************************************
# Function:     display_and_log_message()
# Description:  Log a message to LOGFILE and to the screen.
# Arguments:    $1 = message to be logged.
# Return Value: None.
# **************************************************************************
function display_and_log_message
{
  printf "$1\n" | tee -a $LOGFILE
} # End display_and_log_message()

# **************************************************************************
# Function:     log_message()
# Description:  Log a message and its date only to LOGFILE
# Arguments:    $1 = message to be logged.
# Return Value: None.
# **************************************************************************
function log_message
{
  printf "$1 on `date`\n" >> $LOGFILE
} # End log_message()

# **************************************************************************
# Function:     get_aging_interval()
# Description:  Print out the aging interval if there is one. For what
#               could be an inappropriate action, issue a warning.
# Arguments:    1, 2 or 3
# Return Value: The current PW aging interval (0 if not turned on)
#               If the user appears to be doing an inappropriate action
#               a warning is issued.
# **************************************************************************
function get_aging_interval
{
 if [ "$OSNAME" = "SunOS" ]; then
    CUR_VALUE="`grep MAXWEEKS $PWAGE_FILE | cut -f 2 -d '='`"
 else
    CUR_VALUE=`egrep ^PASS_MAX_DAYS $PWAGE_FILE | awk '{print $2}'`
 fi

 if [ "$CUR_VALUE" = "" ] || [ "$CUR_VALUE" = "99999" ]
 then
   if [ $1 = 2 ]
   #Aging is already off, warn if user trying to re-turn it off
   then
     display "\n***WARNING, it appears that password aging is already off."
     display "   Enter q below if you wish to quit.\n"
   elif [ $1 = 3 ]
   then
     display "\n***WARNING, it appears that password aging is not turned on."
     display "   To turn it on enter the aging interval or q to quit.\n"
   fi
   return 0 && continue
 else
   if [ $1 = 1 ]
   then
     #Aging is already on, warn if user and give current interval
     display "\n***WARNING, it appears that password aging is already on"
     if [ "$OSNAME" = "SunOS" ]; then
        display "   and aging at the rate of $CUR_VALUE week(s)."
     else
        CUR_WEEKS=`expr $CUR_VALUE \/ 7`
        display "   and aging at the rate of $CUR_WEEKS week(s)."
     fi
     display "   To change, enter an interval or q to quit.\n"
   elif [ $1 = 3 ]
   then
     if [ "$OSNAME" = "SunOS" ]; then
        display "Passwords are currently expiring every $CUR_VALUE week(s)."
     else
        CUR_WEEKS=`expr $CUR_VALUE \/ 7`
        display "Passwords are currently expiring every $CUR_WEEKS week(s)."
     fi
   fi
 fi
 if [ "$OSNAME" = "SunOS" ]; then
    return $CUR_VALUE
 else
    return $CUR_WEEKS
 fi
} # End get_aging_interval

# **************************************************************************
# Function: check_lock()
# Description:  Checks to see if LOCKFILE exists; if so see if age_pw
#               process exists. If so, exit with message.
# Arguments:    None.
# Return Value: None.
# **************************************************************************
function check_lock
{
  # Verify if the age_pw lock file exists, if so warn user then exit.
  if [ -r $LOCKFILE ]; then
    PS="`ps -ef| grep age_pw | grep -v grep | grep -v $$`"
    if [ "$PS" != "" ]; then
      display "** WARNING:"
      display "** Only one user may run age_pw at one time."
      display "** If you are certain nobody else is running the command,"
      display "** please run \" rm $LOCKFILE\"."
      cleanup_exit 0
    fi
  else
    touch $LOCKFILE
  fi
} # end check_lock ()

# **************************************************************************
# Function: modify_policy_file()
# Description:  Modifies ${PWAGE_FILE}, the Solaris or Linux file used to set up
#               password aging when new users are created.
# Arguments:    One: number of MAXWEEKS to use; if "" is argument,
#               password aging is turned off. (WARNWEEKS is invalid if
#               aging is turned off.)
# Return Value: None.
# **************************************************************************
function modify_policy_file
{
  # set up password policy file (used when new users are created)
  printf "Modifying $PWAGE_FILE policy file...\n" >> $LOGFILE
  if [ "$OSNAME" = "SunOS" ]; then
     egrep -v "MAXWEEKS|WARNWEEKS" $PWAGE_FILE > $TMPFILE
     echo "MAXWEEKS=$1" >> $TMPFILE
     echo "WARNWEEKS=1" >> $TMPFILE
     chmod 644 $PWAGE_FILE
     mv $TMPFILE $PWAGE_FILE
     chmod 444 $PWAGE_FILE
  else
     if [ "$1" = "" ]; then
        DAYS=99999
     else
        DAYS=`expr $1 \* 7`
     fi
     egrep -v "Password aging controls modified|^PASS_MAX_DAYS|^PASS_WARN_AGE" \
                $PWAGE_FILE > $TMPFILE
     printf "# Password aging controls modified `date`\n" >> $TMPFILE
     printf "PASS_MAX_DAYS      $DAYS\n" >> $TMPFILE
     printf "PASS_WARN_AGE      7\n" >> $TMPFILE
     chmod 644 $PWAGE_FILE
     cp $TMPFILE $PWAGE_FILE
  fi
} # End modify_policy_file ()

# **************************************************************************
# Function: age_existing_users()
# Description:  For existing users, users are aged at the number of
#               weeks specified in first argument, excluding users in
#               ${EXCLUDED_USERS}. Or aging is turned off for all
#               cms users (${EXCLUDED_USERS is irrelevant in this case).
# Arguments:    One: number of MAXWEEKS to use; if value is -1, then
#               password aging will be turned off.
# Return Value: None.
# **************************************************************************
function age_existing_users
{
# create a list of cms users
cat ${ETCPASSWD} | grep '/usr/bin/cms' | cut -d":" -f1 > $TMPFILE

if [ $1 != "-1" ]
then
  printf "Changing password aging for cms users...excluding:\n" >> $LOGFILE
  for i in `cat $EXCLUDED_USERS`
  do
    # See if user to exclude exists in /etc/passwd and is a cms user
    grep "^${i}" /etc/passwd | grep '/usr/bin/cms' > /dev/null 2>&1
    if [ $? = 0 ]
    then
      printf "${i}\n" >> $LOGFILE
    fi
  done
  MAXDAYS=`expr $1 \* 7`
  for user in `cat $TMPFILE | egrep -v -f ${EXCLUDED_USERS}`
  do
    if [ "$OSNAME" = "SunOS" ]; then
       passwd -x ${MAXDAYS} -w ${WARNDAYS} $user || \
        display_and_log_message "ERROR: 'passwd' command generated error for user $user--call Services"
    else
       chage -m 0 -M ${MAXDAYS} $user || \
        display_and_log_message "ERROR: 'chage' command generated error for user $user--call Services"
       # This outputs info similar to Solaris (passwd -x)
       printf "password information changed for user $user\n"
    fi
  done
#  log_message "Turning on or changing password aging for users:"
else
  for user in `cat $TMPFILE`
  do
    if [ "$OSNAME" = "SunOS" ]; then
       passwd -x -1 $user || \
        display_and_log_message "ERROR: 'passwd' command generated error for user $user--call Services"
    else
       chage -m 0 -M 99999 $user || \
        display_and_log_message "ERROR: 'chage' command generated error for user $user--call Services"
       # This outputs info similar to Solaris (passwd -x)
       printf "password information changed for user $user\n"
    fi
  done
#  log_message "Turning off password aging for users:"
fi

# ljd 6/10/02 The number of cms users can be extensive and fill up
#             the log file. Therefore, no longer do it.
#cat $TMPFILE >> $LOGFILE

}

# **************************************************************************
# Function: cleanup_exit()
# Description:  Removes all temporary files and exits with argument.
# Arguments:    One: exit code
# Return Value: None.
# **************************************************************************
function cleanup_exit
{
  rm -f $LOCKFILE
  rm -f $TMPFILE
  exit $1
}
# **************************************************************************
# MAIN PROGRAM
# **************************************************************************

# ----------------
# GLOBAL CONSTANTS
# ----------------
if [ "$OSNAME" = "SunOS" ]; then
   readonly PWAGE_FILE="/etc/default/passwd"
else
   readonly PWAGE_FILE="/etc/login.defs"
fi
readonly ETCPASSWD="/etc/passwd"
readonly ETCPASSWDSAVE="/cms/tmp/passwd"
readonly NSSWITCH="/etc/nsswitch.conf"
readonly EXCLUDED_USERS="/cms/db/age_pw_exclude"
readonly PWAGE_TEMPLATE="/cms/toolsbin/age_pw_template"
readonly LOGFILE="/cms/install/logdir/admin.log"
readonly LOCKFILE=/cms/tmp/age_pw.lock
readonly TMPFILE=/cms/tmp/passwd.tmp.$$
readonly DEF_WEEKS=9
readonly WARNDAYS=7
readonly true=1
# Error codes
readonly NO_LOGFILE=1
readonly NOT_ROOT=2
readonly NO_EXCL=3
readonly DIR_SERV_USED=4

# -----------------------------------------------
# GLOBAL_VARIABLES - sorted in alphabetical order
# -----------------------------------------------

# --------------------------
# Check that admin.log exists
# --------------------------
if [ ! -w $LOGFILE ]
then
  printf "age_pw: ERROR: $LOGFILE does not exist--CMS not installed?\n"
  cleanup_exit $NO_LOGFILE
fi

# --------------------------
# Verify if the user is root
# --------------------------
if [ $LOGNAME != root ]
then
  display_and_log_message "\nage_pw: ERROR: You must be root to run this command!\n"
  cleanup_exit $NOT_ROOT
fi

# --------------------------
# Check that $EXCLUDE_USERS exists
# and cleanup_exit if it does not.
# --------------------------
if [ ! -f $EXCLUDED_USERS ]
then
  display_and_log_message \
        "\nage_pw: ERROR: Required file, $EXCLUDED_USERS does not exist--exiting. Call Services.\n"
  cleanup_exit $NO_EXCL
fi

# --------------------------
# Remove any blank lines in the
# file (possible fat finger
# by customers or services).
# --------------------------
. /olds/olds-funcs
remove_blank_lines $EXCLUDED_USERS

# --------------------------
# Check that NIS/NIS+ or LDAP
#  is used and cleanup_exit if it does.
# --------------------------
DIR_USED="`grep '^passwd:' $NSSWITCH | egrep '(nis|db|ldap)'`"
if [ "${DIR_USED}" != ""  ]
then
  display_and_log_message \
       "\nERROR: Password aging cannot be implemented on systems using NIS, NIS+ or LDAP.\n"
  cleanup_exit $DIR_SERV_USED
fi

check_lock
log_message "** Starting age_pw"

# If there is no password policy file, copy the default in
if [ ! -f $PWAGE_FILE ]
then
   cp $PWAGE_TEMPLATE $PWAGE_FILE
   chmod 444 $PWAGE_FILE
fi

PS3=" 1) Turn on password aging\n 2) Turn off password aging\n 3) Change password aging interval\n    or q to quit: "
check_for_123 RESPONSE
case $RESPONSE in
      1|3)
       get_aging_interval $RESPONSE
       PS3="Enter Maximum number of week(s) before passwords expire"
       check_range AGE_WEEKS
       modify_policy_file $AGE_WEEKS
       age_existing_users $AGE_WEEKS
       display_and_log_message  "Expiring passwords every $AGE_WEEKS week(s)"
       ;;
      2)
       get_aging_interval $RESPONSE
       PS3="Turn off password aging for all CMS users"
       check_for_yorn YORN
       if [ $YORN = "y" ]
       then
         display_and_log_message "Turning off password aging for all CMS users"
         modify_policy_file ""
         age_existing_users -1
       else
        display_and_log_message "Not turning off password aging for all CMS users"
       fi
       ;;
esac

log_message "** Exiting age_pw"

cleanup_exit 0
```

### Файл /cms/toolsbin/awk_rtdb

```bash title="/cms/toolsbin/awk_rtdb"
$3=="|" {
    print "CHG",$1,$2,$3,$5
}
$1==">" {
    print "ADD",$2,$3
}
$3=="<" {
    print "DEL",$1,$2
}
```

### Скрипт меню cmssvc
Ищем реальное расположение команды:

```bash
$ which cmssvc
/usr/bin/cmssvc
```

На самом деле это скрипт */usr/bin/cmssvc*

```bash title="/usr/bin/cmssvc"
#!/bin/ksh
# set PATH to find any possible command
CMSBASE=`cat /usr/bin/cms_base`
PATH=$CMSBASE/install/bin:$CMSBASE/install/cms_install:$CMSBASE/bin:$PATH export PATH

. adm_func
. /opt/informix/bin/setenv

ids_run_status
if [ $? -eq 4 ]
then
    if [ "`uname`" = SunOS ]; then
        echo "cmssvc: Warning IDS off-line. IDS can be turned on"
        echo "with the run_ids command on the cmssvc menu."
    else
        echo "cmssvc: Warning IDS off-line.  It will take approx 45 seconds to"
        echo "start cmssvc.  IDS can be turned on with the run_ids command on"
        echo "the cmssvc menu."
    fi
fi

export INFORMIXCONTIME=1
exec ins_proc -l $CMSBASE/install/logdir/admin.log -B
```

Смотрим что за переменная CMSBASE
```bash
# cat /usr/bin/cms_base
/cms
```

Смотрим что в первом упоминаемом каталоге PATH:
```bash
# ls /cms/install/bin
adm_func         config_firewall  ins_proc    restart_firewall   turn_off_cms
aom_start        converter        java_fips   restore            turn_off_ids
back_all         db_backup        patch_rmv   restrict_dbaccess  turn_on_cms
backup           db_restore       pkgrm       run_cms            turn_on_ids
compress_backup  firewall         postremove  run_ids            uncompress_backup
config_fips      firewall_status  preremove   stop_firewall
```

Смотрим скрипт *adm_func* (включенные функции), выполняемый командой cmssvc:

```bash title="/cms/install/bin/adm_func"
# cat 

#!/bin/ksh
################################################################################
#
# Copyright Avaya Inc., All Rights Reserved.
#
# THIS IS UNPUBLISHED PROPRIETARY SOURCE CODE OF Avaya Inc.
#
# The copyright notice above does not evidence any actual or intended
# publication of such source code.
#
# Some third-party source code components may have  been modified from their
# original versions by Avaya Inc.
#
# The modifications are Copyright Avaya Inc., All Rights Reserved.
#
#############################################################################
#
# Shell functions used for CMS administration
#
# MODIFICATIONS:
#
# DATE        WHO   DESCRIPTION
# 24/07/09    hgl   wi00330195: Changes to remove software mirroring
# 09/22/09    ctb   wi00341160: Changes for Tivoli and onbar (64 bit)
# 09/22/10    ctb   wi00730810: add make_label for cmsadm backup to non-tape
# 10/05/10    ctb   wi00733837: add make_maint_label for maint backups
# 10/21/10    ctb   wi00825053: change date format for make_label
# 11/10/10    ctb   wi00830455: add check_for_stor_slice for LAN maint bkups
# 12/03/10    ctb   wi00838310: Change rotate_bkup_res_log() to handle bigger
#                               size and more files
# 03/28/11    hgl   wi00868445: Make changes to support LSI RAID card
# 04/20/11    ctb   wi00877847: Fix add_uid for 386 for R16.3
# 04/20/11    gps   wi00878382: Limit IDS memory to 85% of physical memory
# 03/08/12    ctb   wi00989256: Changes for R17 - mostly echo -> printf
# 06/21/12    ctb   wi01018226: Turn off IDS should not remove it from upstart
# 08/08/12    hgl   wi01028744: Linux ODBC connection does not work
# 10/04/12    hgl   wi01049418: create RAID controller type check function
# 10/24/12    hgl   wi01054724: Linux tape support
# 11/15/12    hgl   wi01059695: Change CMS GID to 1001 on Linux
# 10/02/13    hgl   wi01118671: Support adding disks for VMware CMS
# 09/19/14    hgl   wi01189368: Only VMware disks are allowed to add to IDS
# 10/07/14    hgl   wi01193149: Add HP support
# 04/14/15    ctb   CMS-505:    add default for $TIMEOUT in ids_on
# 05/07/15    hgl   CMS-535:    Add Dell R220 support
# 06/18/15    hgl   CMS-582:    skip disks with filesystem when add_disks
# 08/08/15    hgl   CMS-478:    move search and exclusive lists from backup to
#                               adm_func so that netbackup can use them
# 04/15/16    sww   CMS-1400    Add support for HPE DL20 G9 platform
# 05/18/16    jba   CMS-1342    Put in dependency check ids.conf or turn_on_ids
#                               for Linux systems
# 03/23/17    jba   CMS-1488    IDS doesn't come up in case iso get mounted in the linux system
#                               after system gets rebooted post baseload upgrade
# 06/21/17   gyeh   CMS-1716    use groupadd to create a user group
# 09/18/17   gyeh   CMS-1722    firewall support: allow quit for menu,
#                               allow default for y/n
# 10/05/17        ctb   CMS-1762:   Support for FIPS mode on/off: setting ciphers
#                                                           in /etc/ssh/sshd_config
# 01/17/18    ctb   CMS-1840:   Fix modify_sshconfig to match requirements
################################################################################

CMSOS=`uname`
export CMSOS

# make_menu <heading> <items>
# items are separated by space or newline(\n). if an item has spaces, use @#
# to escape
# e.g. make_menu "Select one of the following" "Turn@#on@#IDS Turn@#off@#IDS"
# if % is part of item use %%% to escape it.
# e.g. make_menu "Select one of the following" "Turn@#on@#IDS get%%%%val"
make_menu()
{
        NUM_ITEMS=`printf "$2" | wc -w | sed "s,[       ],,g"`

        # for w in $2 does not work for Linux if $2 has newline char
        # This is the case for multiple tape drives
        ARG2=`printf "$2"`

        PROMPT=$1

        n=1
        for w in $ARG2
        do
                # wi01054724 use @# as escape characters instead of %
                # space was replaced by % for items have spaces since space
                # is delimiter for "for w in ...". printf uses % as format char
                # use @# instead
                w="`echo $w | sed 's,@#, ,g'`"
                PROMPT="$PROMPT\n  $n) $w"
                n=`expr $n + 1`
        done

        if [ "$ALLOW_QUIT" = "1" ]
        then
               PROMPT="$PROMPT\nEnter choice (1-$NUM_ITEMS) or q to quit: "
        else
               PROMPT="$PROMPT\nEnter choice (1-$NUM_ITEMS): "
        fi
}

# usage get_nth index item1 item2 ... itemN
get_nth()
{
        shift $1
        echo $1
}

# usage menu_select <prompt> <min> <max> [ <tracefile> [ <default flag> ]]
menu_select()
{
        while [ 1 ]
        do
                if [ "$5" = "y" -a $2 -eq $3 -a "$INTERACTIVE" != "0" ]
                then
                        REPLY=$2
                else
                        printf "\n$1"
                        read REPLY
                fi

                if [ "$ALLOW_QUIT" = "1" -a "$REPLY" = "q" ]
                then
                        break;
                fi


                printf ${REPLY} 2>/dev/null | egrep '^[-+]?[0-9]+$' > /dev/null
                if [ $? -eq 0 -a "$REPLY" -ge $2 -a "$REPLY" -le $3 ]
                then
                        break
                else
                        if [ "$INTERACTIVE" = "0" ]
                        then
                                printf "$1\nREPLY=$REPLY\nselection failed" >> $LOGFILE
                                REPLY=$2
                                BATCHERR=1
                                break
                        fi
                fi
        done
        if [ -n "$4" ]
        then
                printf "$1" | sed 's/^/# /' >> $4
                printf $REPLY >> $4
        fi
}

get_yn()
{
        while [ 1 ]
        do
                printf "\n$1"
                read REPLY

                if [ "$REPLY" = "" -a "$DEFAULT_VAL" != "" ]
                then
                     REPLY=$DEFAULT_VAL;
                     break;
                fi

                case $REPLY in
                y|yes|Y|YES|Yes )  REPLY=y; break;;
                n|no|N|NO|No )  REPLY=n; break;;
                esac
                if [ "$INTERACTIVE" = "0" ]
                then
                        printf "$1\nREPLY=$REPLY\nselection failed" >> $LOGFILE
                        REPLY=y
                        BATCHERR=1
                        break
                fi
        done
        if [ -n "$2" ]
        then
                printf "$1" | sed 's/^/# /' >> $2
                echo $REPLY >> $2
        fi
}

get_nstr()
{
        REPLY=""
        until [ "$REPLY" ]
        do
                printf "\n$1"
                read REPLY
        done
        if [ -n "$2" ]
        then
                echo "$1" | sed 's/^/# /' >> $2
                echo $REPLY >> $2
        fi
}

# cms_run_status - returns 0 if cms running, 1 otherwise
cms_run_status()
{
      if [ "$CMSOS" = "Linux" ]; then
          if [ -e /etc/init/cms.conf ]
          then
              RESULT=`/sbin/initctl status cms | grep -c "running" 2>/dev/null`
              if [ $RESULT -eq 1 ]
              then
                 return 0
              else
                 return 1
              fi
          else
                 return 1
          fi

       else
          egrep "^cm:.*respawn" /etc/inittab >/dev/null
       fi
}

#usage ck_cms_off <errmsg>
ck_cms_off()
{
        cms_run_status
        if [ $? -eq 0 ]
        then
                printf "$1\n"
                exit 1
        fi
}

#usage rm_file_entries <file> "<grep string>"
rm_file_entries()
{
        grep -v "$2" $1 >/tmp/rmf$$
        cp /tmp/rmf$$ $1
        rm /tmp/rmf$$
}

# add an entry to inittab file
#usage add_itab_entry <file> "<entry>"
add_itab_entry()
{
        id=`echo "$2" | cut -d: -f1`
        OLDENTRY=`grep "^$id:" $1`
        [ "$OLDENTRY" = "$2" ] && return 0
        [ -n "$OLDENTRY" ] &&   rm_file_entries $1 "^$id:"
        echo "$2" >>$1
}

stop_unix_log()
{
        rm_file_entries /etc/inittab "^lo:12345:respawn:/etc/logit >/dev/sysmsg 2>&1"
        /etc/init q
        printf "Stopping UNIX log ... "
        RC=0
        while [ $RC = 0 ]
        do
                sleep 5
                ps -e | grep 'logit'
                RC=$?
        done
        echo "done"
}

start_unix_log()
{
        add_itab_entry /etc/inittab "lo:12345:respawn:/etc/logit >/dev/sysmsg 2>&1"
        /etc/init q
}

#usage: set_cms_version "<version>"
#requires environment from autoconfig file
set_cms_version()
{
    VERSTMP=/tmp/ver$$
    for i in `grep -l "$PKGNAME" /usr/options/*.name /usr/lib/installed/CONTENTS 2>/dev/null`
    do
        sed "s/${PKGNAME}.*(.*)/${PKGNAME} ($VERSION)/" <$i >$VERSTMP &&
        mv $VERSTMP $i
    done
}

#get_gid sets the GID variable to group id, group is created if required
#usage: get_gid <group name>
get_gid()
{
    G_ENTRY=`grep "^$1:" /etc/group`
    if [ $? -ne 0 ]
    then
        echo "Creating $1 group id"
        if [ "$CMSOS" = "SunOS" ]; then
            GID=200
        else
            GID=1001
        fi
        while grep ":$GID:" /etc/group >/dev/null
        do
                GID=`expr $GID + 1`
        done
        groupadd -g "$GID" $1
    else
        GID=`echo $G_ENTRY | cut -d: -f3`
    fi
}

#usage: add_uid <user name> <desc> <group name>
# wi00877847: Removed sun/i386 test - use the same code for both platforms
add_uid()
{
    get_gid $3
    grep "^$1:.*:$GID:.*:/export/home/$1:" /etc/passwd >/dev/null
    if [ $? -ne 0 ]
    then
        userdel -d $1 2>/dev/null

        echo "Creating $1 user id"
        rm -fr /export/home/$1

        if [ ! -d /export/home ]
        then
                mkdir /export/home
        fi

        # set the default home directory
        useradd -D -b /export/home > /dev/null

        # add the user, create the directory
        useradd -g $3 -s /usr/bin/ksh -m -c $2 $1
        echo "Assigning a new password for $1"
        passwd $1
    fi
}

#usage: ids_run_status
ids_run_status()
{
# # # # # # # # # # # # # # # # # # # # # # # # # # #
#
#   Status              Return Value
#   ------------        --------------
#
#   On-Line                   0
#   Shutting Down             1
#   Off-Line                  2
#   Unknown                   3
#   Down                      4
#   Quiescent                 5
#   Recovery                  6
#   Initialization            7
#
# # # # # # # # # # # # # # # # # # # # # # # # # # #

   ts0="On-Line"
   ts1="Shutting Down"
   ts2="Off-Line"
   ts4="shared memory not initialized"
   ts5="Quiescent"
   ts6="Recovery"
   ts7="Initializing"

# Write the onstat output to a temporary file since multiple
# calls of onstat would take too much time.

   tmpfile="/tmp/idstmp.$$"
   onstat > ${tmpfile}

if [ `cat ${tmpfile} | grep -wic "${ts0}"` -gt 0 ]; then rt=0

      elif [ `cat ${tmpfile} | grep -wic "${ts1}"` -gt 0 ]; then rt=1

      elif [ `cat ${tmpfile} | grep -wic "${ts2}"` -gt 0 ]; then rt=2

      elif [ `cat ${tmpfile} | grep -wic "${ts4}"` -gt 0 ]; then rt=4

      elif [ `cat ${tmpfile} | grep -wic "${ts5}"` -gt 0 ]; then rt=5

      elif [ `cat ${tmpfile} | grep -wic "${ts6}"` -gt 0 ]; then rt=6

      elif [ `cat ${tmpfile} | grep -wic "${ts7}"` -gt 0 ]; then rt=7

      else rt=3

   fi

   if [ -f ${tmpfile} ]; then rm -f /tmp/idstmp.$$; fi
   return ${rt}
}

#usage: ids_on
ids_on()
{
   . /opt/informix/bin/setenv
   CMSBASE=`cat /usr/bin/cms_base`
   PATH=$CMSBASE/install/bin:$PATH export PATH

   : ${TIMEOUT:=600}

   echo "Please wait for initialization"
   oninit &
   printf ". "

   ((count=0))
   ids_run_status
   RTN=$?

   while [ $RTN -ne 0 -a $count -lt $TIMEOUT ]; do
      sleep 3
      ((count=$count+3))
      printf ". "
      ids_run_status
      RTN=$?
   done

   if [ $count -lt $TIMEOUT ]; then
      printf "\n***** IDS is now up *****\n"
   else
      if [ $RTN -ne 6 ]; then
         printf "IDS could not be started.  Try shutting IDS down and try again.\n"
      else
         printf "IDS Server is Recovering.\nPlease wait until IDS is on-line. \n"
      fi
   fi

#  Make sure ids exists in the inittab file
   if [ "$CMSOS" = "SunOS" ]; then
      if [ `grep -wc 'id:0234:once:$CMSBASE/install/bin/turn_on_ids' /etc/inittab` -eq 0 ]; then
         ID_ITAB_ENTRY="id:0234:once:$CMSBASE/install/bin/turn_on_ids >/dev/null 2>&1"
         add_itab_entry /etc/inittab "$ID_ITAB_ENTRY"
      fi
   fi
}

#usage: ids_off
# Note: turning off IDS does not remove it from inittab (Sun) or upstart (Linux)
# Therefore, IDS will always start on a reboot.
ids_off()
{
   printf "\n*** Turning off IDS, Please wait  ***\n"; printf ". "
   printf ". "; onmode -ky;  printf ". "
   printf "\n\n*** IDS is now off ***\n"
}

#usage ck_ids_off <errmsg>
ck_ids_off()
{
   ids_run_status
   if [ $? -ne 4 ]; then
      if [ $# -eq 1 ]; then printf "$1\n"; fi
      exit 1
   fi
}

device_check()
{
   if [ -z "$1" ]; then
      echo "Creation of the raw device failed: device detection failed by $2 command"
      exit 1
   else
      echo "Device $1 detected by $2 command"
   fi
}

ids_Linux_setup()
{
   raw -q /dev/raw/raw1 > /dev/null 2>&1
   if [ $? -ne 0 ]; then
      dev=`sfdisk -s | grep -v ^total | grep -v loop | sort | head -1 | awk -F: '{print $1}'`
      device_check $dev 'sfdisk'
      infdev=`fdisk -l | grep $dev | tail -1 | awk '{print $1}'`
      device_check $infdev 'fdisk'
      raw /dev/raw/raw1 $infdev > /dev/null 2>&1
      if [ $? -eq 0 ]; then
         echo "Raw device created on $infdev"
      else
         echo "Raw device creation failed"
         exit 1
      fi
      sleep 1
      rm -f /cmsdisk
      ln -s /dev/raw/raw1 /cmsdisk
   fi
   chown informix:informix /dev/raw/raw1 > /dev/null 2>&1
   chmod 660 /dev/raw/raw1
   rm -f /INFORMIXTMP/* > /dev/null 2>&1

   # VMware CMS may have more than one disk
   dmidecode -s system-manufacturer 2>/dev/null | grep VMware > /dev/null
   if [ $? -eq 0 ]; then
      # all other disks are for Informix
      j=1
      for disk in `sfdisk -s | grep -v ^total | grep -v loop | sort | awk -F: '{print $1}'`
      do
         # skip boot disk
         if [ $j -eq 1 ]; then
            j=`expr $j + 1`
            continue
         fi

         # /sys/block/sda/device/vendor has vendor information
         grep -iq VMware /sys/block/${disk##*/}/device/vendor
         if [ $? -ne 0 ]; then
            continue
         else
            # Skip disks with file system. A disk may be added on a different
            # datastore for backup
            blkid | grep ${disk} > /dev/null
            if [ $? -eq 0 ]; then
               continue
            fi
         fi
         raw -q /dev/raw/raw${j} > /dev/null 2>&1
         if [ $? -ne 0 ]; then
            raw /dev/raw/raw${j} $disk > /dev/null 2>&1
            sleep 1
         fi
         chown informix:informix /dev/raw/raw${j}
         chmod 660 /dev/raw/raw${j}
         j=`expr $j + 1`
      done
   fi
}

ids_Linux_network_check()
{
    // Add check for whether or not the network is set up before turning on IDS
    SECS=20
    NETUP=`ip link | grep BROADCAST | grep UP | wc -l`
    if [ $NETUP -ne 1 ]
    then
        echo "`date +%R:%S`  Network is not ready yet. IDS will wait up to $SECS seconds" >> $INFORMIXDIR/cmsids.log
        #A 20 second timeout for checking the network
        for S in {1..$SECS}
        do
                NETUP=`ip link | grep BROADCAST | grep UP | wc -l`
                if [ $NETUP -eq 0 ]
                then
                        sleep 1
                else
                        break
                fi
        done
    fi
    if [ $NETUP -eq 0 ]
    then
      echo "`date +%R:%S`  Network not ready yet but IDS will start anyway" >> $INFORMIXDIR/cmsids.log
    else
      echo "`date +%R:%S`  Network is ready and IDS will start" >> $INFORMIXDIR/cmsids.log
    fi

}


rotate_log()
{
    # LOGSIZE and LOGFILE should be set before call rotate_log
    # LOGSIZE in blocks, 512 bytes per block. Default LOGSIZE is 15MB
    if [ "$LOGSIZE" = "" ]; then
        LOGSIZE=30000
    fi
    if [ "$LOGFILE" = "" ]; then
        return
    fi

    # Rotate the log - allow for 3 logs @ $LOGSIZE each
    unset i x
    find $LOGFILE -size +$LOGSIZE 2>/dev/null |\
    egrep "$LOGFILE" >/dev/null 2>&1
    if [ $? -eq 0 ]; then
        x=02
        for i in 02 01
        do
            if [ -f $LOGFILE.$i ]
            then
               cp -p $LOGFILE.$i $LOGFILE.$x 2>/dev/null
            fi
            x=$i
        done
        cp -p $LOGFILE $LOGFILE.01 2>/dev/null

        > $LOGFILE
    fi
    unset i x
}

# cmsadm/LAN backup related function
rotate_bkup_res_log()
{
    # LOGFILE is set by calling script
    rotate_log
}
rotate_dsm_logs()
{
    export DSM_LOG=/cms/install/logdir
    export DSMI_LOG=/cms/install/logdir
    export DSMI_DIR=/opt/tivoli/tsm/client/api/bin64
    export DSMI_INF_DIR=/opt/tivoli/tsm/client/informix/bin64
    export DSMI_CONFIG=/opt/tivoli/tsm/client/api/bin64/dsm.opt

    # keep error logs less than 150K, LOGSIZE in blocks, 512 bytes per block
    LOGSIZE=300

    LOGFILE=$DSM_LOG/dsmerror.log
    rotate_log

    LOGFILE=$DSMI_LOG/dsierror.log
    rotate_log

    if [ -r /opt/informix/bin/setenv ]; then
        . /opt/informix/bin/setenv
        BAR_LOG=`grep "^BAR_ACT_LOG" $INFORMIXDIR/etc/onconfig.cms \
        | awk '{print $2}'`
        LOGFILE=$BAR_LOG
        rotate_log
    fi
}
rotate_nbu_logs()
{
    export NBU_LOG=/usr/openv/netbackup/logs

    # NetBackup logs moved to /usr/openv/netbackup/logs
    # Remove log rotation for nbuerror.log and nbuierror.log
    # See /usr/openv/netbackup/logs/README.debug
    # Turn on NetBackup logging
    for log_list in bpbackup bprestore bpcd bpbkar infxbsa
    do
        if [ ! -d $NBU_LOG/$log_list ]; then
            mkdir -p $NBU_LOG/$log_list
            chmod 1777 $NBU_LOG/$log_list
        fi
    done

    # Cleanup logs older than 7 days
    find $NBU_LOG -name "log.*" -mtime +7 -exec rm -f {} \;

    if [ -r /opt/informix/bin/setenv ]; then
        . /opt/informix/bin/setenv
        BAR_LOG=`grep "^BAR_ACT_LOG" $INFORMIXDIR/etc/onconfig.cms \
        | awk '{print $2}'`
        LOGFILE=$BAR_LOG
        rotate_log
    fi
}
convert()
{
    # cms11020128 - backup the RSC admin files
    # make the rsc change in the convert() function for both backup.SVR4.sh and backup.tivoli.sh
    # to pick up the change during the cms adm backup
    RD=/usr/platform/`uname -i`/rsc
    RSCDIR=`echo ${RD}`
    [ -f ${RSCDIR}/rscadm.usershow.settings ] && cp -p ${RSCDIR}/rscadm.usershow.settings \
    ${RSCDIR}/rscadm.usershow.settings.old
    [ -f ${RSCDIR}/rscadm.show.settings ] && cp -p ${RSCDIR}/rscadm.show.settings \
    ${RSCDIR}/rscadm.show.settings.old

    CMSBASE=`cat /usr/bin/cms_base`
    INSTALLDIR=$CMSBASE/install/cms_install
    LOGFILE=${CMSBASE}/install/logdir/backup.log

    cdir=`pwd`
    mkdir /tmp/conv.$$; cd /tmp/conv.$$
    if [ -f ${CMSBASE}/install/bin/converter ]; then
        ${CMSBASE}/install/bin/converter -l $LOGFILE 1>/dev/null 2>&1
        if [ $? -eq 0 ]; then
            # MR 130876 do not use mv, use cp to preserve ownership
            [ -f ${INSTALLDIR}/storage.def ] && cp ${INSTALLDIR}/storage.def \
            ${INSTALLDIR}/storage.def.old
            [ -f ${INSTALLDIR}/cms.install ] && cp ${INSTALLDIR}/cms.install \
            ${INSTALLDIR}/cms.install.old
            [ -f ${INSTALLDIR}/fp.install ] && cp ${INSTALLDIR}/fp.install \
            ${INSTALLDIR}/fp.install.old
            [ -f stor.def ] && {
                cp stor.def ${INSTALLDIR}/storage.def
                rm stor.def
            }
            [ -f setup.out ] && {
                cp setup.out ${INSTALLDIR}/cms.install
                rm setup.out
            }
            [ -f fp.install ] &&  {
                cp fp.install ${INSTALLDIR}/fp.install
                rm fp.install
            }
       fi
    fi
    cd $cdir; rm -rf /tmp/conv.$$
}

tune_onconfig()
{
# when preinstall calls tune_onconfig cms_base does not exist
CMSBASE=`cat /usr/bin/cms_base 2>/dev/null`
if [ "$CMSBASE" != "" ]; then
    ADMLOG=${CMSBASE}/install/logdir/admin.log
else
    ADMLOG=`tty`
fi

# Enabling View Folding to Improve Query Performance
# Make sure outer join will not use a temporary table
IFX_FOLDVIEW=1
export IFX_FOLDVIEW

ONCFG=/opt/informix/etc/onconfig.cms
if [ ! -f $ONCFG ]; then
    echo "There is no "$ONCFG
    exit 1
fi

# oninit fails if DBSERVERNAME has ".", "-" characters
# Invalid character(s) in DBSERVERNAME or DBSERVERALIASES (cms_emu.dr.avaya.com).
shortname=`hostname | awk -F. '{print $1}' | sed /-/s//_/g`
grep "cms_${shortname}" $ONCFG > /dev/null
if [ $? -ne 0 ]; then
  echo "`date` Adding cms_${shortname} to $ONCFG" >> $ADMLOG
  sed -e "/^DBSERVERALIASES/s/.*/DBSERVERALIASES oacms_ol,cms_net,cms_${shortname}/" \
         $ONCFG > /tmp/onconfig.$$
  cp /tmp/onconfig.$$ $ONCFG
  rm /tmp/onconfig.$$
fi

# see /opt/informix/etc/onconfig.std for detailed tunable information
if [ "$CMSOS" = "SunOS" ]; then
   numprocs=`/usr/sbin/psrinfo | wc -l | sed s/" "//g`
else
   numprocs=`cat /proc/cpuinfo | grep ^processor -c`
fi
if [ $numprocs -gt 3 ]; then
    numprocs=`expr $numprocs - 1`
fi
if [ $numprocs -gt 1 ]; then
    # MultiPROCESSOR
    F1=`grep "^VPCLASS" $ONCFG | awk -F',' '{print $2}' \
         | awk -F'=' '{print $2}' | awk '{print $1}'`
    if [ -z "$F1" ]; then
        echo "`date` ERROR: VPCLASS parameter is not set in $ONCFG" | tee -a $ADMLOG
    elif [ $F1 -ne $numprocs ]; then
        echo "`date` Modifying VPCLASS in $ONCFG" >> $ADMLOG
        sed -e "/^MULTIPROCESSOR/s/0/1/" \
            -e "/^VPCLASS/s/num=[0-9]*/num=$numprocs/" \
            $ONCFG > /tmp/onconfig.tuned
        cp /tmp/onconfig.tuned $ONCFG
    fi
fi

# Limit the amount of physical memory IDS can use to 85% of memory
if [ "$CMSOS" = "SunOS" ]; then
   # prtconf memory in MB
   mem=`prtconf | grep "Memory size" | awk '{print $3}'`
   mem=`expr $mem \* 1024`
else
   # meminfo in KB
   mem=`grep "MemTotal" /proc/meminfo  | awk '{print $2}'`
fi
multiplier=85
currSHMTOTVal=`egrep \^SHMTOTAL $ONCFG`
currSHMTOTAL=`egrep \^SHMTOTAL $ONCFG | awk '{ print $2 }'`
newMemSetting=`expr $mem \* $multiplier / 100`

if [ "$newMemSetting" != "$currSHMTOTAL" ]; then
    sed s/"^$currSHMTOTVal"/"SHMTOTAL $newMemSetting"/g $ONCFG >/tmp/o$$
    cp -p $ONCFG $ONCFG.OLD
    cp /tmp/o$$ $ONCFG
    rm /tmp/o$$
    echo "`date` Changed SHMTOTAL setting in" >> $ADMLOG
    echo "   $ONCFG from $currSHMTOTAL to $newMemSetting" >> $ADMLOG
fi

# Tune DS_MAX_QUERIES, DS_TOTAL_MEMORY, DS_NONPDQ_QUERY_MEM
# DS_NONPDQ_QUERY_MEM: DS_TOTAL_MEMORY/4
# convert to KB
dsmem=`expr $mem / 2`
non_pdq_mem=`expr $dsmem / 4`
F2=`grep "^DS_TOTAL_MEMORY" $ONCFG | awk '{print $2}'`
if [ -z "$F2" ]; then
    echo "`date` ERROR: DS_TOTAL_MEMORY parameter is not set in $ONCFG" | tee -a $ADMLOG
elif [ "$F2" != $dsmem ]; then
    echo "`date` Modifying DS_TOTAL_MEMORY in $ONCFG" >> $ADMLOG
    sed -e "/^DS_TOTAL_MEMORY/s/$F2/$dsmem/" \
        -e "/^DS_NONPDQ_QUERY_MEM/s/DS_NONPDQ_QUERY_MEM [0-9]*/DS_NONPDQ_QUERY_MEM $non_pdq_mem/" \
        $ONCFG > /tmp/onconfig.tuned
    cp /tmp/onconfig.tuned $ONCFG
fi

# 1/4 physical memory by number of 8k pages
bufmem=`expr $mem / 32`
F1=`grep "^BUFFERPOOL.*size=8" $ONCFG | awk -F',' '{print $2}' \
      | awk -F'=' '{print $2}' | awk '{print $1}'`
if [ -z "$F1" ]; then
    echo "`date` ERROR: BUFFERPOOL default is not set in $ONCFG" | tee -a $ADMLOG
elif [ "$F1" != $bufmem ]; then
    echo "`date` Modifying 8K BUFFERPOOL to $bufmem in $ONCFG" >> $ADMLOG
    sed -e "/^BUFFERPOOL.*size=8.*buffers/s/buffers=[0-9]*/buffers=$bufmem/" \
        -e "/^BUFFERPOOL.*size=8.*buffers/s/lrus=[0-9]*/lrus=64/" \
        -e "/^BUFFERPOOL.*size=8.*buffers/s/lru_min_dirty=[0-9]*\.[0-9]*/lru_min_dirty=50/" \
        -e "/^BUFFERPOOL.*size=8.*buffers/s/lru_max_dirty=[0-9]*\.[0-9]*/lru_max_dirty=60/" \
        $ONCFG > /tmp/onconfig.tuned
    cp /tmp/onconfig.tuned $ONCFG
fi
rm -f /tmp/onconfig.tuned

F1=`grep "^BUFFERPOOL.*size=2" $ONCFG | awk -F',' '{print $2}' \
      | awk -F'=' '{print $2}' | awk '{print $1}'`
if [ -z "$F1" ]; then
    echo "`date` ERROR: 2K BUFFERPOOL is not set in $ONCFG" | tee -a $ADMLOG
elif [ "$F1" != "12800" ]; then
    echo "`date` Modifying 2K BUFFERPOOL to 12800 in $ONCFG" >> $ADMLOG
    sed -e "/^BUFFERPOOL.*size=2.*buffers/s/buffers=[0-9]*/buffers=12800/" \
        -e "/^BUFFERPOOL.*size=2.*buffers/s/lrus=[0-9]*/lrus=64/" \
        -e "/^BUFFERPOOL.*size=2.*buffers/s/lru_min_dirty=[0-9]*\.[0-9]*/lru_min_dirty=50/" \
        -e "/^BUFFERPOOL.*size=2.*buffers/s/lru_max_dirty=[0-9]*\.[0-9]*/lru_max_dirty=60/" \
        $ONCFG > /tmp/onconfig.tuned
    cp /tmp/onconfig.tuned $ONCFG
fi

# only look at the first NETTYPE
F2=`grep "^NETTYPE" $ONCFG | head -1 | awk '{print $2}'`
if [ -z "$F2" ]; then
    echo "`date` ERROR: NETTYPE parameter is not set in $ONCFG" | tee -a $ADMLOG
elif [ "$F2" != "onipcstr,2,1000,CPU" ]; then
    echo "`date` Modifying NETTYPE in $ONCFG" >> $ADMLOG
    sed -e "/^NETTYPE/s/$F2/onipcstr,2,1000,CPU/" \
        $ONCFG > /tmp/onconfig.tuned
    cp /tmp/onconfig.tuned $ONCFG
fi
rm -f /tmp/onconfig.tuned

if [ "$CMSOS" = "SunOS" ]; then
   nettype="ontlitcp"
else
   nettype="onsoctcp"
fi

# sqlhosts related changes
SQLHOSTS=/opt/informix/etc/sqlhosts
grep "cms_net $nettype `uname -n` 50000" $SQLHOSTS > /dev/null
if [ $? -ne 0 ]; then
  echo "`date` Adding cms_net to $SQLHOSTS" >> $ADMLOG
  grep "50000" $SQLHOSTS  > /tmp/sqlhosts.$$
  if [ $? -eq 0 ]; then
    echo "`date` 50000 port existing in $SQLHOSTS. Removing it" >> $ADMLOG
    cp -p $SQLHOSTS /tmp/sqlhosts.$$
    grep -v "50000" /tmp/sqlhosts.$$ > $SQLHOSTS
    rm /tmp/sqlhosts.$$
  fi
  echo "cms_net $nettype `uname -n` 50000" >> $SQLHOSTS
fi

grep "cms_${shortname} $nettype `uname -n` 50001" $SQLHOSTS > /dev/null
if [ $? -ne 0 ]; then
  echo "`date` Adding cms_${shortname} to $SQLHOSTS" >> $ADMLOG
  grep "50001" $SQLHOSTS  > /tmp/sqlhosts.$$
  if [ $? -eq 0 ]; then
    echo "`date` 50001 port existing in $SQLHOSTS. Removing it" >> $ADMLOG
    cp -p $SQLHOSTS /tmp/sqlhosts.$$
    grep -v "50001" /tmp/sqlhosts.$$ > $SQLHOSTS
    rm /tmp/sqlhosts.$$
  fi
  echo "cms_${shortname} $nettype `uname -n` 50001" >> $SQLHOSTS
fi
}

# wi00730810: Used by cmsadm backup to create a file name as a tape label
make_label()
{
    MACHINE_NAME=`uname -n`
    DATE=`date +%y%m%d%H%M%S`

    BK_LABEL=CMSADM-$VERSION-$DATE-$MACHINE_NAME
}

# wi00733837: backup file for maintenance backups needs host name
make_maint_label()
{
    hostname=$1

    [ $hostname ] && nchars=`echo $hostname | wc -m | awk ' { x = $1 - 1;  printf("%2.2d", x); } '`
    if [ $nchars -gt 99 -o "$hostname" = "" ]
    then
       pad_host=unknown00000000000
       nchars=15
    fi
    if [ $nchars -lt 15 ]
    then
       fill=`expr 15 - $nchars`
       pad_host=`echo $hostname $fill | awk ' { printf("%s%0*d", $1, $2, 0); }'`
    else
       nchars=15
       pad_host=`echo $hostname | awk ' { x = substr($1, 1, 15); print(x); }'`
    fi

    echo "${nchars}-${pad_host}"

}

# wi00830455: Used by LAN MAINT backups & restores to check for /storage slice
check_for_stor_slice()
{
  if [ "$CMSOS" = SunOS ]; then
    printf "disk\n0\nquit\n" > /tmp/fmtcmd.$$
    DISK=`format -f /tmp/fmtcmd.$$ 2>&1 | grep selecting | awk '{print $2}'`
    if [ -z "$DISK" ]; then
      echo "Can't find boot disk through format command" >> $LOGFILE
      rm -f /tmp/fmtcmd.$$
      exit 1
    fi
    rm -f /tmp/fmtcmd.$$
    # Do we have the bigger disk that will have /storage?
    disk_size=`get_phys_disk_size`
    if [ $disk_size -lt 256000 ]; then
      return 1
    fi

    # Is /storage slice 5?
    stor_slice=`df | grep "^/storage" | awk  '{print $2 }' | awk -F'/' '{ print $4 }'`
    if [ -z "$stor_slice" -o "$stor_slice" != "${DISK}s5" ];then
      return 1
    fi
  fi
  return 0
}

# check RAID card type
check_card_type()
{
    ARCCONF=/opt/StorMan/arcconf
    if [ "$CMSOS" = SunOS ]; then
      MEGACLI=/opt/MegaRAID/CLI/MegaCli
    else
      MEGACLI=/opt/MegaRAID/MegaCli/MegaCli64
    fi
    HPSSACLI=/usr/sbin/hpssacli

    if [ "$CMSOS" = SunOS ]; then
        # getconfig 1 AD outputs: Controllers found: 1
        adcnt=`$ARCCONF getconfig 1 AD 2>/dev/null | grep "Controllers found" \
               | awk '{print $NF}'`
        rm -f UcliEvt.log
        if [ "$adcnt" -eq 1 ]; then
            RAIDTYPE="Intel"
            return 0
        fi
    fi
    # MegaCli -adpCount retuns number of adaptors
    # command not found returns 127. Assume we don't have more than 9 controllers
    $MEGACLI -adpCount -NoLog > /dev/null 2>&1
    if [ $? -gt 0 -a $? -lt 10 ]; then
        RAIDTYPE="LSI"
        return 0
    fi
    # check if this is an HP system
    dmidecode -s system-manufacturer 2>/dev/null | grep ^HP$ > /dev/null
    if [ $? -eq 0 ]; then
# Check if this is the DL20 G9. It only has 1 disk and no RAID
        if [ `dmidecode -s system-product-name | grep DL20 | wc -l` -eq 1 ]; then
          RAIDTYPE="DL20"
          return 0
        fi
        ctlnum=`$HPSSACLI ctrl all show 2>&1 | grep -i slot | wc -l`
        slot=`$HPSSACLI ctrl all show | grep -i slot | head -1| awk '{print $6}'`
        if [ $ctlnum -ge 1 ]; then
            RAIDTYPE="HPSSA"
            return 0
        fi
    fi
    # check if this is a VMware CMS
    dmidecode -s system-manufacturer 2>/dev/null | grep VMware > /dev/null
    if [ $? -eq 0 ]; then
        RAIDTYPE="VMware"
        return 0
    fi
    # check if this is a Dell R220
    # the OEM string is 05E5 from common hardware team
    dmidecode -t 11 2>/dev/null | grep "\[05E5\]" > /dev/null
    if [ $? -eq 0 ]; then
        RAIDTYPE="R220"
        return 0
    fi

    return 1
}

# wi00868445: Make changes to support LSI RAID card
# same code in restore for Solaris. Make sure to change both places
# return disk size in MB
get_phys_disk_size()
{
    if [ "$LOGFILE" = "" ]; then
        LOGFILE=/dev/null
    fi

    check_card_type
    if [ $? -eq 1 ]; then
        echo 1
        return 1
    fi

    if [ "$RAIDTYPE" = "Intel" ]; then
        echo "Intel chip RAID card. arcconf utility will be used" >> $LOGFILE
        # outputs:
        # Size                               : 140009 MB
        # check if there are different size disks
        size1=`$ARCCONF getconfig 1 PD | grep Size | awk '{print $3}' | sort -n | head -1 | awk -F'.' '{print $1}'`
        size2=`$ARCCONF getconfig 1 PD | grep Size | awk '{print $3}' | sort -n | tail -1 | awk -F'.' '{print $1}'`
        if [ `expr $size2 - $size1` -gt 2048 ]; then
            echo "Disk sizes are different. You can use $ARCCONF to check disk size." >> $LOGFILE
            echo "e.g. $ARCCONF getconfig 1 PD | grep Size" >> $LOGFILE
            echo "All disks should have same size" >> $LOGFILE
            echo 1
            return 1
        fi
        pdisk_sz=`$ARCCONF getconfig 1 PD | grep Size | awk '{print $3}' | sort -n | head -1 | awk -F'.' '{print $1}'`
        rm -f UcliEvt.log
    elif [ "$RAIDTYPE" = "LSI" ]; then
        echo "LSI chip RAID card. MegaCli utility will be used" >> $LOGFILE
        # outputs:
        # Raw Size: 279.396 GB [0x22ecb25c Sectors]
        # check if there are different size disks
        size1=`$MEGACLI -PDList -a0 -NoLog | grep "Raw Size" | awk '{print $3}' | sort -n | head -1 | awk -F'.' '{print $1}'`
        size2=`$MEGACLI -PDList -a0 -NoLog | grep "Raw Size" | awk '{print $3}' | sort -n | tail -1 | awk -F'.' '{print $1}'`
        # convert to MB
        size1=`expr $size1 \* 1024`; size2=`expr $size2 \* 1024`
        if [ `expr $size2 - $size1` -gt 2048 ]; then
            echo "Disk sizes are different. You can use $MEGACLI to check disk size." >> $LOGFILE
            echo "e.g. $MEGACLI -PDList -a0 -NoLog | grep \"Raw Size\"" >> $LOGFILE
            echo "All disks should have same size" >> $LOGFILE
            echo 1
            return 1
        fi
        pdisk_sz=`$MEGACLI -PDList -a0 -NoLog | grep "Raw Size" | awk '{print $3}' | sort -n | head -1 | awk -F'.' '{print $1}'`
        pdisk_sz=`expr $pdisk_sz \* 1024`
    elif [ "$RAIDTYPE" = "HPSSA" ]; then
        echo "HP Smart Storage Array. hpssacli utility will be used" >> $LOGFILE
        # physicaldrive 1I:2:1 (port 1I:box 2:bay 1, 300 GB): OK
        # physicaldrive 1I:2:1 (port 1I:box 2:bay 1, 73.4 GB): Failed
        size1=`$HPSSACLI ctrl slot=$slot pd all show status | grep -i physicaldrive | awk '{print $7}' | sort -n  | head -1 | awk -F'.' '{print $1}'`
        size2=`$HPSSACLI ctrl slot=$slot pd all show status | grep -i physicaldrive | awk '{print $7}' | sort -n  | tail -1 | awk -F'.' '{print $1}'`
        if [ `expr $size2 - $size1` -gt 2 ]; then
            echo "Disk sizes are different. You can use $HPSSACLI to check disk size." >> $LOGFILE
            echo "e.g. $HPSSACLI ctrl slot=$slot pd all show status" >> $LOGFILE
            echo "All disks should have same size" >> $LOGFILE
            echo 1
            return 1
        fi
        pdisk_sz=`expr $size1 \* 1024`
    else
        echo "Unable to find correct RAID card" >> $LOGFILE
        echo 1
        return 1
    fi
    echo $pdisk_sz
}

# wi00830247: check for any removable media, return mountpoint
# formatted as "^mountpoint".
check_for_removable_media()
{
    EXCL_RM_MEDIA=""
    CMSDEV=`df -k / | grep -v Filesystem | awk '{print $1}' | sed 's/[0-9]$//'`
    if [ "$CMSOS" = SunOS ]; then
        # on Solaris x86 /lib/libc.so.1 is mounted as lofs, remove it from exclude list
        # /usr/lib/libc/libc_hwcap1.so.1 - /lib/libc.so.1 lofs - no
        RM_DEVICE=`mount -p | grep -v "^$CMSDEV" | awk '{print $3}' | sed 's/^\//^/' | egrep -v libc.so`
    else
        # file systems othet than OS and CMS
        RM_DEVICE=`mount -l | grep -v "^$CMSDEV" | awk '{print $3}' | sed 's/^\//^/'`
    fi
    for MOUNTPOINT in $RM_DEVICE
    do
        if [ -n "$EXCL_RM_MEDIA" ]
        then
             EXCL_RM_MEDIA="$EXCL_RM_MEDIA|$MOUNTPOINT"
        else
             EXCL_RM_MEDIA=$MOUNTPOINT
        fi
    done
}

# CMS-478: move search and exclusive lists from backup to adm_func so that
# netbackup can use them
get_search_path()
{
    # determine the list of root entries were going to work on
    # this way we don't have to scan all the way down the directory just
    # to determine we don't want the files
    # wi00830247 - add rmdisk (USB mounted storage device)
    # wi01044642 - remove nfs, removable media mount points from search list to
    # speed up find. We make find not to descend directories on other filesystems.
    # Just in case add $EXCL_RM_MEDIA to $EXCL_LIST too
    SRCH_EXCL_LIST="^proc|^cdrom|^net|^tmp|^core|^vol|^floppy|^xfn|^rmdisk"
    check_for_removable_media
    [ -n "$EXCL_RM_MEDIA" ] && SRCH_EXCL_LIST="$SRCH_EXCL_LIST|$EXCL_RM_MEDIA"
    cd / > /dev/null
    SRCHLIST=`ls -1A | egrep -v $SRCH_EXCL_LIST`

    # wi00342204: Add export/home to SRCHLIST. It is its
    # own mount point so isn't picked up in the find -mount.
    SRCHLIST="$SRCHLIST export/home"
    export SRCHLIST

    #
    # create exclusive lists
    #
    if [ "$CMSOS" = SunOS ]; then
        # SOLARIS exclude file/directory list
        #
        # MR v9cms020024 - remove ^cms/db/gem/c_custom/(.*), ^cms/db/gem/h_custom/(.*)
        # and ^cms/db/gem/r_custom/(.*) from exclude list
        # MR cms11020178 - add BI/add_on/data/forwarder/(.*) for OA buffer file directory
        # MR 148615 - exclude /tmp
        # MR wi00730810  - exclude /storage for R16.2
        #
        EXCL_LIST="^var/tmp|^dump/tmp|^etc/saf/zsmon/_pmpipe\
|^cms/install/bin/restore|^etc/default/init|^etc/release\
|^etc/saf/zsmon/_pid|^etc/saf/_sacpipe|^etc/saf/_cmdpipe\
|^etc/mnttab|^etc/initpipe|^etc/syslog.pid|^etc/nologin\
|^etc/.name_service_door|^var/spool/lp/temp|^var/spool/lp/tmp\
|^var/spool/lp/requests|^var/spool/(.*)mqueue/(.*)|^var/spool/locks/(.*)\
|^var/spool/uucp/dummyech/Z/(.*)|^dev|^devices|^usr/dbtemp/(.*)\
|^usr/lib/cms/Aname|^usr/lib/cms/Pname|^usr/lib/cms/Sname\
|^cms/cmstables/(.*)|^cms/db/inf/cms.dbs|^cms/db/journal/shortcut/(.*)\
|^cms/db/journal/timetable/(.*)|^cms/pbx/master|^cms/pbx/sim_pbx/(.*)\
|^cms/pbx/acd(.*)/(.*)|^cms/tmp/(.*)|^cms/dc/chr/chr_log\
|BI/add_on/data/forwarder/(.*)|^tmp|^storage\
|^INFORMIXTMP|^var/spool/cups/tmp|^var/spool/postfix/pid\
|^cms/net_mgmt/spirit/LogTail_(.*)|^cms/net_mgmt/spirit/alarm.log(.*).ptr\
|^cms/net_mgmt/spirit/config/alarm.log(.*).ptr|^cms/net_mgmt/log/(.*)\
|^cms/net_mgmt/spirit/logging/(.*)|^cms/net_mgmt/spirit/persist/(.*)"

        # If a file size changing too fast during backup, cpio can not catch the file
        # state and will fail. This exclude list is for files that are needed as space
        # holder and their contents are not important.
        EXCL_CHECK_LIST="\"var/adm/(.*)\"|\"var/spool/(.*)\"\
|\"var/cron/log\"|\"opt/informix/cmsids.log\"|\"cms/toolsbin/logfile\"\
|\"cms/env/cms_mon/proc_log\"|\"cms/aas/vector/(.*)\""

        # swap files to exclude
        EXCL_SWAP=`swap -l 2>/dev/null | sed '1d' | awk '{print "^"$1}'\
           | cut -c1,3- | paste -s -d"|" -`
    else
        # LINUX exclude file/directory list
        #
        # MR v9cms020024 - remove ^cms/db/gem/c_custom/(.*), ^cms/db/gem/h_custom/(.*)
        # and ^cms/db/gem/r_custom/(.*) from exclude list
        # MR cms11020178 - add BI/add_on/data/forwarder/(.*) for OA buffer file directory
        # MR 148615 - exclude /tmp
        # MR wi00730810  - exclude /storage for R16.2
        #
        EXCL_LIST="^var/tmp|^dump/tmp\
|^cms/install/bin/restore|^etc/system-release\
|^etc/mtab|^etc/nologin\
|^etc/.name_service_door|^var/spool/cups/tmp|^var/spool/cups/temp\
|^var/spool/cups/requests|^var/spool/(.*)mqueue/(.*)|^var/lock/(.*)\
|^var/spool/uucp/dummyech/Z/(.*)|^dev|^usr/dbtemp/(.*)\
|^usr/lib/cms/Aname|^usr/lib/cms/Pname|^usr/lib/cms/Sname\
|^cms/cmstables/(.*)|^cms/db/inf/cms.dbs|^cms/db/journal/shortcut/(.*)\
|^cms/db/journal/timetable/(.*)|^cms/pbx/master|^cms/pbx/sim_pbx/(.*)\
|^cms/pbx/acd(.*)/(.*)|^cms/tmp/(.*)|^cms/dc/chr/chr_log\
|BI/add_on/data/forwarder/(.*)|^tmp|^storage\
|^INFORMIXTMP|^var/spool/postfix/pid\
|^cms/net_mgmt/spirit/LogTail_(.*)|^cms/net_mgmt/spirit/alarm.log(.*).ptr\
|^cms/net_mgmt/spirit/config/alarm.log(.*).ptr|^cms/net_mgmt/log/(.*)\
|^cms/net_mgmt/spirit/logging/(.*)|^cms/net_mgmt/spirit/persist/(.*)\
|^opt/avaya/SAL/gateway/GatewayUI/logging/(.*)\
|^opt/avaya/SAL/gateway/Gateway/xGate.log\
|^opt/avaya/SAL/gateway/KeystoreUtility/logging/(.*)\
|^opt/avaya/SAL/gateway/SpiritAgent/persist/(.*)\
|^opt/avaya/SAL/gateway/SpiritAgent/logging/(.*)\
|^opt/avaya/SAL/gateway/SALWatchdog/pids/(.*)\
|^opt/avaya/SAL/gateway/SALWatchdog/logging/(.*)\
|^opt/avaya/SAL/gateway/SALWatchdog/bin/mgmt/logs/(.*)\
|^opt/avaya/SAL/gateway/logging/(.*)"

        # If a file size changing too fast during backup, cpio can not catch the file
        # state and will fail. This exclude list is for files that are needed as space
        # holder and their contents are not important.
        EXCL_CHECK_LIST="\"var/log/(.*)\"|\"var/spool/(.*)\"\
|\"var/log/cron(.*)\"|\"opt/informix/cmsids.log\"|\"cms/toolsbin/logfile\"\
|\"cms/env/cms_mon/proc_log\""
        # swap files to exclude
        EXCL_SWAP=`swapon -s 2>/dev/null | sed '1d' | awk '{print "^"$1}'\
          | cut -c1,3- | paste -s -d"|" -`
    fi

    # make sure EXCL_SWAP is not null
    [ -n "$EXCL_SWAP" ] && EXCL_LIST="$EXCL_LIST|$EXCL_SWAP"

    # wi00830247: exclude any removable storage media - this covers
    # a USB-mounted device specified as something other than /rmdisk
    [ -n "$EXCL_RM_MEDIA" ] && EXCL_LIST="$EXCL_LIST|$EXCL_RM_MEDIA"

    EXCL_LIST="$EXCL_LIST|^.autofsck"

    # wi00955226 - Exclude ZFS filessytems from backup
    if [ $CMSOS = SunOS ]; then
        ZFS_EXCL=`zpool list | egrep -v "no pools|ALTROOT$" | cut -d" " -f1`
        for ii in $ZFS_EXCL
        do
            EXCL_LIST="$EXCL_LIST|^$ii"
        done
    fi
    export EXCL_LIST
    cd - > /dev/null
}

# LINUX does not have the ckyorn command that Solaris has in /usr/bin
if [ $CMSOS = Linux ]; then
ckyorn()
{
    help="To respond in the affirmative, enter y, yes, Y, or YES. To respond in\n\
the negative, answer n, no, N, or NO.\n"
    # Process any options
    default=
    Qflag=false
    choices="[y,n,?,q]"
    while [ "$1" ]
    do
         case $1 in
          -Q)
             Qflag=true
            ;;
          -p)
             shift
             prompt=$1
            ;;
          -d)
             shift
             case $1 in
                 y|Y|yes|YES)
                    default=$1
                    ;;
                 n|N|no|NO)
                    default=$1
                    ;;
                 *)
                    default=
                    ;;
             esac
            ;;
           *)
             break
            ;;
         esac
         shift
    done
    if [ $Qflag = "true" ]; then
         choices="[y,n,?]"
    fi

    choice=
    ans=
    while [ ! "$ans" ]
    do
        printf "$prompt $choices ? " > /dev/tty
        read  ans
        if [ "$ans" ]; then
           choice=$ans
        else
           choice=$default
           ans=$default
        fi

        case $choice in
             q|Q|quit|Quit|QUIT)
             if [ $Qflag = true ]; then
                ans=
             fi
             ;;
             y|Y|yes|YES)
             ;;
             n|N|no|NO)
             ;;
             *)
                printf "$help" > /dev/tty
                ans=
             ;;
        esac
    done

    printf $choice

    }

fi

modify_sshdconfig()
{
        echo "Begin modifications to ssh configuration files for security." >> $LOGFILE
        SSHD_CONFIG=/etc/ssh/sshd_config
        SSH_CONFIG=/etc/ssh/ssh_config
        CIPHER_FILE=/cms/install/security/setup_ciphers_def
        MAC_FILE=/cms/install/security/setup_macs_def

    # Add Ciphers to /etc/ssh/sshd_config if they are not there already
    NO_SSH_FILE=
    NO_CIPH_FILE=
    NO_MAC_FILE=
    [ ! -e $SSH_CONFIG ] && {
        echo "Warning: Missing $SSH_CONFIG, cannot set ciphers. You must edit this file manually." >> $LOGFILE
        NO_SSH_FILE=yes
    }
    [ ! -e $CIPHER_FILE ] && {
        echo "Warning: $CIPHER_FILE not found. Check $SSH_CONFIG and add Ciphers manually if necessary." >> $LOGFILE
        NO_CIPH_FILE=yes
    }
    [ ! -e $MAC_FILE ] && {
        echo "Warning: $MAC_FILE not found. Check $SSH_CONFIG and add Macs manually if necessary." >> $LOGFILE
        NO_MAC_FILE=yes
    }
    if [ -z $NO_SSH_FILE ] && [ -z $NO_CIPHER_FILE ] && [ -z $NO_MAC_FILE ];
    then

        # Save a backup
        /bin/cp -bfu $SSH_CONFIG{,.bck}
        /bin/cp -bfu $SSHD_CONFIG{,.bck}
        echo "A copy of $SSH_CONFIG and $SSHD_CONFIG has been made." >> $LOGFILE

        date=`date +%c`

        # If Ciphers and MACs are defined, replace them.  If not, add them.

        grep -v '\(\(Ciphers\)\|\(Macs\)\|\(Avaya\)\)' $SSHD_CONFIG > $SSHD_CONFIG.modified
        sed -s -i -e "2a\
        # Modified $date. Copyright Avaya, Inc., 2018. All Rights Reserved." $SSHD_CONFIG.modified
        sed -s -i -e '/Protocol 2/a\
Ciphers aes128-ctr,aes192-ctr,aes256-ctr\nMACs hmac-sha1,hmac-sha2-256,hmac-sha2-512' $SSHD_CONFIG.modified
        /bin/mv $SSHD_CONFIG.modified $SSHD_CONFIG

        grep -v '\(\(Ciphers\)\|\(Macs\)\|\(MACs\)\|\(Avaya\)\)' $SSH_CONFIG > $SSH_CONFIG.modified
        sed -s -i -e "2a\
        # Modified $date. Copyright Avaya, Inc., 2018. All Rights Reserved." $SSH_CONFIG.modified
        sed -s -i -e '/^Host */a\
        Ciphers aes128-ctr,aes192-ctr,aes256-ctr\n\tMACs hmac-sha1,hmac-sha2-256,hmac-sha2-512' $SSH_CONFIG.modified
        /bin/mv $SSH_CONFIG.modified $SSH_CONFIG

            echo "Modification of $SSH_CONFIG and $SSHD_CONFIG to Ciphers and Macs is complete." >> $LOGFILE
    fi
}
```

### Скрипт сетевых настроек netconfig
 */cms/toolsbin/netconfig*

```bash title="/cms/toolsbin/netconfig"
#!/bin/bash
################################################################################
# Copyright (c) 2016, Avaya. All rights reserved.
#
# NAME: netconfig
#
# DESCRIPTION:
#    Setup network for CMS. It either prompts user to input HOSTNAME, IP address
#    netmask etc or reads from an input file.
#
# Arguments:
#    input_file
#
# Installed:
#    /cms/toolsbin/netconfig
#
# Usage:
#    netconfig <input_file>
#
# MODIFICATIONS:
#
# DATE        WHO   DESCRIPTION
# 12/07/12    hgl   Inital development                             MR wi01063879
# 04/22/14    hgl   Support VM multiple interfaces                 MR wi01166315
# 12/08/14    hgl   Stop Network Manager before configure
#                   the interface                                  MR wi01202254
# 01/13/16    hgl   Support Multi-NICs on different subnets        MR CMS-1280
################################################################################
trap 'err_exit 3' 1 2 3 15

CMSBASE=`cat /usr/bin/cms_base 2>/dev/null`
if [ -z $CMSBASE ]; then
    CMSBASE=/cms
fi
if [ ! -d ${CMSBASE}/install/logdir ]; then
    mkdir -p ${CMSBASE}/install/logdir
fi
LOGFILE=${CMSBASE}/install/logdir/admin.log
if [ ! -d ${CMSBASE}/tmp ]; then
    mkdir -p ${CMSBASE}/tmp
fi
INPUTFILE=${CMSBASE}/tmp/.cms_netconfig
IFDIR=/etc/sysconfig/network-scripts

err_exit ()
{
    if [ $1 -eq 3 ]; then
        echo -e "Caught signal, exiting..." | tee -a $LOGFILE
    fi
    echo -e "Check $LOGFILE for details"
    echo -e "`date` $0 failed\n" | tee -a $LOGFILE
    exit $1
}

usage ()
{
    #
    # Input file has following information
    #
    # Network interface name
    # Interface name
    # Hostname
    # Domain name
    # IP address
    # Network Mask
    # Default Gateway
    # DNS servers
    # Domain search list
    echo "Usage: $0 [INPUT_FILE]"
    echo "e.g. of input file:"
    echo "INTERFACE=eth0"
    echo "HOSTNAME=partridge"
    echo "DOMAIN=dr.avaya.com"
    echo "IPADDR=10.129.149.177"
    echo "NETMASK=255.255.255.0"
    echo "GATEWAY=10.129.149.254"
    echo "DNS1=135.9.1.2"
    echo "DNS2=198.152.7.12"
    echo "SEARCH=\"eng.avaya.com sd.avayay.com\""
    echo "DEFAULTGATEWAY=135.9.131.254"
}

check_multi_NICs ()
{
    # If interfaces are in different subnets need to setup static routing
    > ${CMSBASE}/tmp/.gateway
    for eth in eth0 eth1 eth2 eth3
    do
        gw=`grep ^GATEWAY ${IFDIR}/ifcfg-$eth 2>/dev/null | awk -F= '{print $NF}' | sed 's/                       "//g; s/ //g; /^$/d'`
        if [ -n "$gw" ]; then
            grep -q "$gw" ${CMSBASE}/tmp/.gateway
            if [ $? -ne 0 ]; then
                echo $gw >> ${CMSBASE}/tmp/.gateway
            fi
        fi
    done
    if [ -n "${VALUE[5]}" ]; then
        grep -q "${VALUE[5]}" ${CMSBASE}/tmp/.gateway 2>/dev/null
        if [ $? -ne 0 ]; then
            echo ${VALUE[5]} >> ${CMSBASE}/tmp/.gateway
        fi
    fi
    uniq_gws=`cat ${CMSBASE}/tmp/.gateway | wc -l`
    if [ $uniq_gws -le 1 ]; then
        # The first NIC or NICs with same gateway
        static_routing=no
    else
        static_routing=yes
        gw=""
        for gw1 in `cat ${CMSBASE}/tmp/.gateway`
        do
            if [ -z "$gw1" ]; then
                continue
            fi
            if [ -n "$gw" ]; then
                gw="$gw, $gw1"
            else
                gw="$gw1"
            fi
        done
        NAME[8]="the default gateway from following gateways: $gw"
    fi
}

set_input_default_values ()
{
    ETHLIST=`ifconfig -a | grep ^[a-z] | grep -v lo | while read F1 F2; do echo -e "$F1 \c"                       ; done`
    if [ -z "$ETHLIST" ]; then
        ifconfig -a >> $LOGFILE 2>&1
        echo "Can't find network interface name. The network card" | tee -a $LOGFILE
        echo "is not configured correctly or missing." | tee -a $LOGFILE
        err_exit 1
    fi

    if [ -n "$INTERFACE" ]; then
        VALUE[0]=$INTERFACE
    else
        INTERFACE=`ifconfig -a | grep ^[a-z] | grep -v lo | awk '{print $1}' | sort | head                        -1`
        VALUE[0]=$INTERFACE
    fi
    NAME[0]="the network interface name from following name(s): $ETHLIST(default ${VALUE[0]                       })"

    if [ -n "$HOSTNAME" ]; then
        VALUE[1]=$HOSTNAME
        NAME[1]="the host name of the CMS system (default ${VALUE[1]})"
    else
        NAME[1]="the host name of the CMS system"
    fi
    if [ -n "$DOMAIN" ]; then
        # DOMAIN in ifcfg-ethX includes search domains
        # the same file is shared for input
        # remove search domains from input
        VALUE[2]=`echo $DOMAIN | awk '{print $1}'`
        NAME[2]="the domain name of the CMS system (default ${VALUE[2]})"
    else
        NAME[2]="the domain name of the CMS system"
    fi
    if [ -n "$IPADDR" ]; then
        VALUE[3]=$IPADDR
        NAME[3]="the IP address of the network interface (default ${VALUE[3]})"
    else
        NAME[3]="the IP address of the network interface"
    fi
    if [ -n "$NETMASK" ]; then
        VALUE[4]=$NETMASK
        NAME[4]="the netmask for the subnet of the network interface (default ${VALUE[4]})"
    else
        NAME[4]="the netmask for the subnet of the network interface"
    fi
    if [ -n "$GATEWAY" ]; then
        VALUE[5]=$GATEWAY
        NAME[5]="the gateway for the network interface (default ${VALUE[5]})"
    else
        NAME[5]="the gateway for the network interface"
    fi
    if [ -n "$DNS1" ]; then
        VALUE[6]="$DNS1"
        [ -n "$DNS2" ] && VALUE[6]="$DNS1 $DNS2"
        [ -n "$DNS3" ] && VALUE[6]="$DNS1 $DNS2 $DNS3"
        NAME[6]="the DNS server(s) separated by space (up to three servers) (default ${VALU                       E[6]})"
    else
        NAME[6]="the DNS server(s) separated by space (up to three servers)"
    fi
    if [ -n "$SEARCH" ]; then
        VALUE[7]=$SEARCH
        NAME[7]="the search domains separated by space (default ${VALUE[7]}, \"\" for none)                       "
    else
        NAME[7]="the search domains separated by space (press enter for none)"
    fi
    # VALUE[8] and NAME[8] are for multi-NICs default gateway
    VALUE[8]=""
    NAME[8]=""
    DEFAULTGATEWAY=""
}

get_inputs ()
{
    let i=0
    while (( i<9 ))
    do
        if [ $i -eq 1 ]; then
            # got interface input using this interface history as defaults
            if [ -s ${INPUTFILE}.${VALUE[0]} ]; then
                # ${INPUTFILE}.${VALUE[0]} may have a wrong value
                TMPVAL=${VALUE[0]}
                . ${INPUTFILE}.${VALUE[0]}
                set_input_default_values
                VALUE[0]=$TMPVAL
                /bin/mv ${INPUTFILE}.${VALUE[0]} ${INPUTFILE}.${VALUE[0]}.sv
            fi
        fi
        if [ $i -eq 8 ]; then
            check_multi_NICs
            if [ "$static_routing" != "yes" ]; then
                let i=i+1
                continue
            fi
        fi
        echo -e ""
        echo -e " Enter ${NAME[i]}"
        echo -e ""
        echo -e " ENTER> \c"
        # save default value
        DEFAULT=${VALUE[i]}
        read VAL
        if [ -n "$VAL" ]; then
            if [ "$VAL" == "\"\"" ]; then
                VALUE[i]=""
            else
                VALUE[i]=$VAL
            fi
        fi
        echo -e ""
        echo -e " You have entered [ ${VALUE[i]} ]. Is this correct? (y|n) \c"
        read confirm
        if [ "$confirm" == "y" -o "$confirm" == "Y" ]; then
            let i=i+1
        else
            VALUE[i]=$DEFAULT
        fi
    done

    INTERFACE=${VALUE[0]}
    HOSTNAME=${VALUE[1]%%.*}
    DOMAIN=${VALUE[2]}
    IPADDR=${VALUE[3]}
    NETMASK=${VALUE[4]}
    GATEWAY=${VALUE[5]}
    SEARCH=${VALUE[7]}
    # remove spaces
    if [ -n "${VALUE[6]}" ]; then
        DNS1=`echo ${VALUE[6]} |  awk '{print $1}'`
        DNS2=`echo ${VALUE[6]} |  awk '{print $2}'`
        DNS3=`echo ${VALUE[6]} |  awk '{print $3}'`
    fi
    DEFAULTGATEWAY=${VALUE[8]}
}

display_inputs ()
{
    echo -e "\n  Interface: $INTERFACE"
    echo -e "  CMS Hostname: $HOSTNAME"
    echo -e "  Domainname: $DOMAIN"
    echo -e "  CMS IP address: $IPADDR"
    echo -e "  Netmask: $NETMASK"
    echo -e "  Gateway: $GATEWAY"
    echo -e "  DNS Server1: $DNS1"
    echo -e "  DNS Server2: $DNS2"
    echo -e "  DNS Server3: $DNS3"
    echo -e "  Search domains: $SEARCH"
    if [ -n "$DEFAULTGATEWAY" ]; then
        echo -e "  Default gateway: $DEFAULTGATEWAY\n"
    else
        echo -e " "
    fi
}

echo -e "`date` $0 started" >> $LOGFILE

echo " WARNING: This tool only supports IPv4"

# Running Network Manager may cause ifup failure
# Stop Network Manager and remove it from startup script
# NetworkManager is removed in R18. Just in case
service NetworkManager stop > /dev/null 2>&1
chkconfig NetworkManager off > /dev/null 2>&1

if [ $# -eq 1 ]; then
    if [ ! -e $1 ]; then
        echo "$1 does not exist" | tee -a $LOGFILE
        usage
        err_exit 1
    else
        . $1
    fi
    # make sure one entry for domain
    DOMAIN=`echo $DOMAIN | awk '{print $1}'`
    if [ -z "$INTERFACE" -o -z "$IPADDR" -o -z "$GATEWAY" ]; then
        echo "Wrong input file." | tee -a $LOGFILE
        cat $1 >> $LOGFILE
        usage
        err_exit 1
    fi
    display_inputs
else
    lastinput=`ls -1rt ${INPUTFILE} ${INPUTFILE}.eth[0-9] 2>/dev/null | tail -1`
    if [ -s "$lastinput" ]; then
        . $lastinput
    else
        INTERFACE="" HOSTNAME="" DOMAIN="" SEARCH="" IPADDR="" NETMASK="" GATEWAY=""
        DNS1="" DNS2="" DNS3=""
    fi
    while true
    do
        set_input_default_values
        get_inputs
        display_inputs
        echo -e "  Are the above inputs correct? (y|n) \c"
        read confirm
        if [ "$confirm" == "y" -o "$confirm" == "Y" ]; then
            break
        fi
    done
fi

INPUTFILE=${INPUTFILE}.${INTERFACE}
HWADDR=`ifconfig -a | grep "^$INTERFACE" | awk '{print $NF}'`
UUID=`uuidgen`
# INTERFACE and HOSTNAME are used for remembering user inputs
echo "INTERFACE=\"$INTERFACE\"" > $INPUTFILE
echo "HOSTNAME=\"$HOSTNAME\"" >> $INPUTFILE
# without UUID dbus-send (in network-functions) will generate one
echo "UUID=\"$UUID\"" >> $INPUTFILE
echo "ONBOOT=\"yes\"" >> $INPUTFILE
echo "BOOTPROTO=\"static\"" >> $INPUTFILE
echo "HWADDR=\"$HWADDR\"" >> $INPUTFILE
if [ -n "$SEARCH" ]; then
    echo "$SEARCH" | grep -q "$DOMAIN" 2>/dev/null
    if [ $? -eq 0 ]; then
        echo "DOMAIN=\"$SEARCH\"" >> $INPUTFILE
    else
        echo "DOMAIN=\"$DOMAIN $SEARCH\"" >> $INPUTFILE
    fi
    # SEARCH is only used to remember user input
    echo "SEARCH=\"$SEARCH\"" >> $INPUTFILE
else
    echo "DOMAIN=\"$DOMAIN\"" >> $INPUTFILE
fi
echo "IPADDR=\"$IPADDR\"" >> $INPUTFILE
echo "NETMASK=\"$NETMASK\"" >> $INPUTFILE
echo "GATEWAY=\"$GATEWAY\"" >> $INPUTFILE
if [ -n "$DNS1" ]; then
    echo "DNS1=\"$DNS1\"" >> $INPUTFILE
fi
if [ -n "$DNS2" ]; then
    echo "DNS2=\"$DNS2\"" >> $INPUTFILE
fi
if [ -n "$DNS3" ]; then
    echo "DNS3=\"$DNS3\"" >> $INPUTFILE
fi
if [ -n "$DEFAULTGATEWAY" ]; then
    echo "DEFAULTGATEWAY=\"$DEFAULTGATEWAY\"" >> $INPUTFILE
fi


IFILE=${IFDIR}/ifcfg-${INTERFACE}
OLDIFILE=${CMSBASE}/tmp/.ifcfg-${INTERFACE}
TMPIFILE=${CMSBASE}/tmp/.ifcfg-${INTERFACE}.tmp

# update /etc/sysconfig/network-scripts/ifcfg-ethX
# /etc/resolv.conf /etc/sysconfig/network
if [ -e "$IFILE" ]; then
    # Save the original configuration file to $LOGFILE. If rename to
    # ifcfg-${INTERFACE}.sv nmcli con list uuid could point to this file
    echo "### old $IFILE:" >> $LOGFILE
    cat $IFILE >> $LOGFILE
else
    echo "$IFILE does not exist" >> $LOGFILE
    echo "DEVICE=\"$INTERFACE\"" > $IFILE
    echo "NM_CONTROLLED=\"yes\"" >> $IFILE
    echo "TYPE=\"Ethernet\"" >> $IFILE
    chmod 644 $IFILE
fi
/bin/cp -p $IFILE $OLDIFILE

cat $IFILE | \
egrep -v "^BOOTPROTO|^HWADDR|^DNS|^GATEWAY|^HOSTNAME|^IPADDR|^NETMASK|^ONBOOT|^DOMAIN|^UUID                       " \
> $TMPIFILE

# generate the new interface configuration file
/bin/cp $TMPIFILE $IFILE
cat $INPUTFILE | egrep -v "^INTERFACE|^HOSTNAME|^SEARCH|^DEFAULTGATEWAY" >> $IFILE

echo "### new ${IFILE}:" >> $LOGFILE
cat $IFILE >> $LOGFILE

#
# Active the network
#
# ifup fails can mess up /etc/resolve.conf
# Save and restore /etc/resolve.conf in case ifup fails
/bin/cp -p /etc/resolv.conf /etc/resolv.conf.sv
echo -e "\n Bring the network up. Please wait...\n"
echo -e "`date` ifup $INTERFACE" >> $LOGFILE
ifdown $INTERFACE > /dev/null 2>&1
ifup $INTERFACE >> $LOGFILE 2>&1
rtn=$?
if [ $rtn -ne 0 ]; then
    echo -e "`date` Rerun ifup $INTERFACE" >> $LOGFILE
    # if ifup fails it modifies $IFILE
    /bin/cp $TMPIFILE $IFILE
    cat $INPUTFILE | egrep -v "^INTERFACE|^HOSTNAME|^SEARCH" >> $IFILE
    sleep 3
    # Sometimes ifup returns 0 but network is still down
    ifup $INTERFACE >> $LOGFILE 2>&1
    ping -c 1 -W 1 $GATEWAY >> $LOGFILE 2>&1
    rtn=$?
    if [ $rtn -ne 0 ]; then
       echo -e "`date` ping $GATEWAY failed." >> $LOGFILE
       echo -e "`date` Retry ifdown and ifup to bring the network up." >> $LOGFILE
       ifdown $INTERFACE >> $LOGFILE 2>&1
       ifup $INTERFACE >> $LOGFILE 2>&1
       ping -c 1 -W 1 $GATEWAY >> $LOGFILE 2>&1
       rtn=$?
    fi
fi
if [ $rtn -ne 0 ]; then
    /bin/cp -p $OLDIFILE $IFILE
    /bin/mv /etc/resolv.conf.sv /etc/resolv.conf
    echo -e "Network configuration failed"
    err_exit 1
else
    /bin/rm -f /etc/resolv.conf.sv
fi

# /etc/sysconfig/network
# HOSTNAME=partridge
# NISDOMAIN=dr.avaya.com
hostname $HOSTNAME
grep HOSTNAME /etc/sysconfig/network > /dev/null 2>&1
if [ $? -eq 0 ]; then
    sed -i "/HOSTNAME/s/.*/HOSTNAME=${HOSTNAME}/" /etc/sysconfig/network
else
    echo "HOSTNAME=${HOSTNAME}" >> /etc/sysconfig/network
fi

# add hostname to /etc/hosts
# using hostname-eth0, hostname-eth1 for multi-NICs
# using IP with default gateway as primary entry
# if miti-NICs have same default gateway take the first one for hostname
# in alphabet order (eth0, eth1)
# in case user inputs an empty HOSTNAME
if [ -n "$HOSTNAME" ]; then
    sed -i "/ $HOSTNAME /d"  /etc/hosts
fi
default_host_flag=yes
for eth in eth0 eth1 eth2 eth3
do
    gw=`grep ^GATEWAY ${IFDIR}/ifcfg-$eth 2>/dev/null | awk -F= '{print $NF}' | sed 's/"//g                       ; s/ //g; /^$/d'`
    if [ -z "$gw" ]; then
        continue
    fi
    ipaddr=`grep ^IPADDR ${IFDIR}/ifcfg-$eth 2>/dev/null | awk -F= '{print $NF}' | sed 's/"                       //g; s/ //g; /^$/d'`
    if [ -z "$ipaddr" ]; then
        continue
    fi
    # if DEFAULTGATEWAY is empty grep -q "" returns 0
    echo "$gw" | grep -q "$DEFAULTGATEWAY"
    # the first IP has default gateway
    if [ $? -eq 0 -a "$default_host_flag" == yes ]; then
        default_host_flag=no
        if [ -n "$HOSTNAME" ]; then
            sed -i "/ $HOSTNAME /d"  /etc/hosts
        fi
        echo "$ipaddr $HOSTNAME ${HOSTNAME}.${DOMAIN}" >> /etc/hosts
    else
        sed -i "/ $HOSTNAME-$eth /d"  /etc/hosts
        echo "$ipaddr $HOSTNAME-$eth ${HOSTNAME}-$eth.${DOMAIN}" >> /etc/hosts
    fi
done

domainname $DOMAIN
grep NISDOMAIN /etc/sysconfig/network > /dev/null 2>&1
if [ $? -eq 0 ]; then
    sed -i "/NISDOMAIN/s/.*/NISDOMAIN=${DOMAIN}/" /etc/sysconfig/network
else
    echo "NISDOMAIN=${DOMAIN}" >> /etc/sysconfig/network
fi

#
# setup static routing for Multi-NICs on different subnets
#
if [ -n "$DEFAULTGATEWAY" ]; then
    echo -e "`date` eth interfaces are on different subnets" >> $LOGFILE
    for eth in eth0 eth1 eth2 eth3
    do
        gw=`grep ^GATEWAY ${IFDIR}/ifcfg-$eth 2>/dev/null | awk -F= '{print $NF}' | sed 's/                       "//g; s/ //g; /^$/d'`
        if [ -z "$gw" ]; then
            continue
        fi
        ipaddr=`grep ^IPADDR ${IFDIR}/ifcfg-$eth 2>/dev/null | awk -F= '{print $NF}' | sed                        's/"//g; s/ //g; /^$/d'`
        if [ -z "$ipaddr" ]; then
            continue
        fi
        echo -e "`date` processing $eth IP: $ipaddr Gateway: $gw" >> $LOGFILE
        # create routing table for this gateway
        # reuse routing table name when IP changed
        if [ -s ${IFDIR}/route-$eth ]; then
            echo -e "`date` gateway $gw routing table is already set" >> $LOGFILE
            # each interface should have one routing table ID
            tab=`cat ${IFDIR}/route-$eth | head -1 | awk '{print $NF}'`
            i=${tab: -1}
            # this should not happen, just in case
            grep -q " $tab$" /etc/iproute2/rt_tables
            if [ $? -ne 0 ]; then
                echo "$i CMS${i}" >> /etc/iproute2/rt_tables
            fi
            echo -e "`date` delete and reset the routing table" >> $LOGFILE
            while read route
            do
                echo "ip route del $route" >> $LOGFILE
                ip route del $route > /dev/null 2>&1
            done < ${IFDIR}/route-$eth
        else
            i=`egrep -v "^#|^255|^254|^253" /etc/iproute2/rt_tables | awk '{print $1}' | so                       rt -rn | head -1`
            if [ -z "$i" ]; then
                let i=1
            else
                let i=$i+1
            fi
            echo "$i CMS${i}" >> /etc/iproute2/rt_tables
        fi
        echo -e "`date` ip route add default via $gw dev $eth table CMS${i}" >> $LOGFILE
        ip route add default via $gw dev $eth table CMS${i} >> $LOGFILE 2>&1
        if [ $? -ne 0 ]; then
            echo -e "Adding default gateway $gw for $eth to table CMS${i} failed"
            echo -e "ip route add default via $gw dev $eth table CMS${i} failed" >> $LOGFIL                       E
            err_exit 1
        fi
        # persist the setting for reboot
        echo "default via $gw dev $eth table CMS${i}" > ${IFDIR}/route-$eth

        # setup rules
        if  [ -s ${IFDIR}/rule-$eth ]; then
            echo -e "`date` routing rules for $ipaddr are already set" >> $LOGFILE
            while read tab
            do
                tab=`echo $tab | awk '{print $NF}'`
                ip rule | grep $tab >> $LOGFILE 2>/dev/null
                echo -e "`date` delete and reset the routing rules" >> $LOGFILE
                echo "while ip rule del table $tab 2>/dev/null; do true; done" >> $LOGFILE
                while ip rule del table $tab 2>/dev/null; do true; done
            done < ${IFDIR}/rule-$eth
        fi
        echo -e "`date` ip rule add from ${ipaddr}/32 table CMS${i}" >> $LOGFILE
        ip rule add from ${ipaddr}/32 table CMS${i} >> $LOGFILE 2>&1
        if [ $? -ne 0 ]; then
            echo -e "Adding rule for ${ipaddr} to table CMS${i} failed"
            echo -e "ip rule add from ${ipaddr}/32 table CMS${i} failed" >> $LOGFILE
            err_exit 1
        fi
        # persist the setting for reboot
        echo "from ${ipaddr}/32 table CMS${i}" > ${IFDIR}/rule-$eth
    done

    # set default gateway
    echo -e "`date` Adding default gateway $DEFAULTGATEWAY" >> $LOGFILE
    ip route del default > /dev/null 2>&1
    echo -e "`date` ip route add default via $DEFAULTGATEWAY" >> $LOGFILE
    ip route add default via $DEFAULTGATEWAY >> $LOGFILE 2>&1
    if [ $? -ne 0 ]; then
        echo -e "Adding default gateway $DEFAULTGATEWAY failed"
        echo -e "ip route add default via $DEFAULTGATEWAY failed" >> $LOGFILE
        err_exit 1
    fi
    # persist the setting for reboot
    grep -q "GATEWAY=$DEFAULTGATEWAY" /etc/sysconfig/network 2>/dev/null
    if [ $? -ne 0 ]; then
        sed -i '/^GATEWAY/d' /etc/sysconfig/network
        echo "GATEWAY=$DEFAULTGATEWAY" >> /etc/sysconfig/network
    fi
    # flush cache
    ip route flush cache
fi

echo -e "`date` $0 successfully finished\n" | tee -a $LOGFILE
```
### Переменные окружения
Файл */opt/informix/bin/setenv*, добавляющий переменные окружения

```bash title="/opt/informix/bin/setenv"
# Environment variables for Informix IDS
DB_LOCALE=en_us.utf8;
export DB_LOCALE;
INFORMIXDIR=/opt/informix;
export INFORMIXDIR;
INFORMIXSERVER=cms_ol;
export INFORMIXSERVER;
ONCONFIG=onconfig.cms;
export ONCONFIG;
INFORMIXCONTIME=2
export INFORMIXCONTIME
PATH=$PATH:$INFORMIXDIR/bin;
export PATH;

# 129877: Solaris 9 sets TERMCAP => dbaccess doesn't work
unset TERMCAP;
```

Cписок переменных окружения — env
```bash
(mrc-krl15-ucacms1)-(root)=# env
SPI6=/cms/pbx/acd6/spi.err
SPI7=/cms/pbx/acd7/spi.err
SPI4=/cms/pbx/acd4/spi.err
SPI5=/cms/pbx/acd5/spi.err
DBPATH=/cms/db/inf
HOSTNAME=mrc-krl15-ucacms1
SPI2=/cms/pbx/acd2/spi.err
SELINUX_ROLE_REQUESTED=
SPI3=/cms/pbx/acd3/spi.err
TERM=xterm
SHELL=/bin/bash
SPI1=/cms/pbx/acd1/spi.err
HISTSIZE=1000
SSH_CLIENT=10.0.45.51 50181 22
INFORMIXDIR=/opt/informix
SELINUX_USE_CURRENT_RANGE=
SPI8=/cms/pbx/acd8/spi.err
SSH_TTY=/dev/pts/0
USER=root
LD_LIBRARY_PATH=:/opt/informix/lib:/opt/informix/lib/esql
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;05;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arj=01;31:*.taz=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lz=01;31:*.xz=01;31:*.bz2=01;31:*.tbz=01;31:*.tbz2=01;31:*.bz=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.rar=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.axv=01;35:*.anx=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.mpc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.axa=01;36:*.oga=01;36:*.spx=01;36:*.xspf=01;36:
U=/usr/spool/uucppublic
ONCONFIG=onconfig.cms
INFORMIXTERM=terminfo
MAIL=/var/spool/mail/root
PATH=/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin:usr/informix/bin:/opt/informix/bin:
PWD=/root
INFORMIXCONTIME=2
LANG=en_US.UTF-8
PS1=\n(mrc-krl15-ucacms1)-(root)=#
SELINUX_LEVEL_REQUESTED=
DB_LOCALE=en_us.utf8
HISTCONTROL=ignoredups
INFORMIXSERVER=cms_ol
SHLVL=1
HOME=/root
TERMINFO=/cms/terminfo
LOGNAME=root
SSH_CONNECTION=10.0.45.51 50181 172.21.103.17 22
LESSOPEN=||/usr/bin/lesspipe.sh %s
G_BROKEN_FILENAMES=1
_=/bin/env
```

Посмотреть значение конкретной переменной:

```bash
# echo $PATH
/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin:usr/informix/bin:/opt/informix/bin:/opt/informix/bin:/opt/informix/bin

# env | grep INFORMIX
INFORMIXDIR=/opt/informix
INFORMIXTERM=terminfo
INFORMIXCONTIME=2
INFORMIXSERVER=cms_ol
```

### Скрипт создания базы данных
/opt/Informix/bin/dbinit.sh

```bash title="/opt/Informix/bin/dbinit.sh"
#!/bin/ksh
################################################################################
# Copyright (c) 2015, Avaya. All rights reserved.
#
# NAME: dbinit.sh
#
# DESCRIPTION:
#    Initialize CMS database and create dbspaces for CMS database
#    CMS R16 only has two data dbspaces cmsdbs and aasdbs
#    aasdbs for visual vector is dropped in R17
#
# WARNING: Adding disk relies on the sequence of disks. When a disk is added
#          as data disk or backup disk you can't drop it or change it.
#
# Arguments:
#    None
#
# Installed:
#    on CMS DVD and /opt/informix/bin
#
# Usage:
#    dbinit.sh [add_disks]
#
# MODIFICATIONS:
#
# DATE        WHO   DESCRIPTION
# 06/14/09    hgl   Inital development for CMS R16
# 09/22/11    hgl   wi00939992: In case disk geometry changes, leave 1GB on s7
#                               unused for onbar and ontape binary restore.
# 03/15/12    hgl   wi00990624: Add Linux support. Remove aasdbs since visual
#                               vector is no longer supported.
# 08/08/12    hgl   wi01028744: Linux ODBC connection does not work. Configure
#                               DBSERVERALIASES
# 10/02/13    hgl   wi01118671: Repartition disk and/or add disks for VMware CMS
# 09/19/14    hgl   wi01189368: Only VMware disks are allowed to add to IDS
# 02/13/15    hgl   wi01160841: set new IDS 12 FULL_DISK_INIT parameter to 1
# 06/18/15    hgl   CMS-582: skip disks with filesystem when add_disks
################################################################################

trap 'err_exit 3' 1 2 3 15

LOGFILE=/opt/informix_install.log

err_exit ()
{
  if [ $1 -eq 3 ]; then
    printf "Caught signal, exiting...\n" | tee -a $LOGFILE
  fi
  printf "Please check $LOGFILE and /opt/informix/cmsids.log for details\n"
  printf "Please fix the problem and re-run $0\n"
  printf "`date` Creating CMS database failed\n\n" | tee -a $LOGFILE
  exit $1
}

repartition_disk ()
{

  # You can use following commands to get original disk partition creating
  # scripts.
  # grep "^# parted" /root/cms_kickstart.log | sed '/^# /s///g'

  # check if the size changed or not
  newsize=`parted $cmsdev unit s print | grep "$cmsdev" | awk '{print$3}'`
  let newsize=${newsize%s}-1
  oldsize=`parted $cmsdev unit s print | grep "extended" | awk '{print$3}'`
  let oldsize=${oldsize%s}

  printf "`date` The new disk size: ${newsize} sectors\n" | tee -a $LOGFILE
  printf "`date` The old disk size: ${oldsize} sectors\n" | tee -a $LOGFILE

  if [ `expr "$newsize" - 2097152` -gt "$oldsize" ]; then
    printf "`date` Repartition the disk for large OVA configuration\n" | tee -a $LOGFILE
  else
    printf "`date` No need to repartition the disk\n" | tee -a $LOGFILE
    return
  fi

  # repartition the disk to increase disk size
  # Number  Start     End          Size         Type      File system     Flags
  # 1       2048s     1169400s     1167353s     primary   ext4            boot
  # 4       43112448s 1258291199s  1215178752s  extended                  lba
  # ...
  # F1      F2        F3           F4           F5        F6

  # parted skips master boot record
  # [root@cms~]# parted -a cylinder -s /dev/sda unit kB mkpart primary 0 600445
  # Error: You requested a partition from 0.00kB to 600445kB.
  # The closest location we can manage is 32.3kB to 600445kB.

  > /root/parted_old_size.sh
  > /root/parted_new_size.sh
  nump=`fdisk -l $cmsdev | grep "^/dev/" | wc -l`
  if [ "$nump" -ne 11 ]; then
    printf "`date` Wrong number of CMS disk partitions: $nump. It should be 11.\n" | tee -a $LOGFILE
    printf "`date` Repartition the disk failed\n" | tee -a $LOGFILE
    err_exit 1
  fi
  parted $cmsdev unit s print | grep "[0-9]" | tail -$nump | \
  while read F1 F2 F3 F4 F5 F6
  do
    F2=${F2%s}; F3=${F3%s}
    if [ "$F1" -eq 4 -o "$F1" -eq "$nump" ]; then
      NF3=$newsize
    else
      NF3=$F3
    fi
    if [ "$F1" -lt 4 ]; then
      echo "# parted -a opt -s $cmsdev unit s mkpart $F5 $F2 $F3" >> /root/parted_old_size.sh
      echo "# parted -a opt -s $cmsdev unit s mkpart $F5 $F2 $NF3" >> /root/parted_new_size.sh
    else
      echo "parted -a opt -s $cmsdev unit s mkpart $F5 $F2 $F3" >> /root/parted_old_size.sh
      echo "parted -a opt -s $cmsdev unit s mkpart $F5 $F2 $NF3" >> /root/parted_new_size.sh
    fi
  done

  chmod 755 /root/parted_old_size.sh /root/parted_new_size.sh
  cp -p /root/parted_old_size.sh /root/parted_old_size_`date "+%Y%m%d_%T"`.sh

  #
  # delete extended partition. parted does not allow to delete mounted partitions
  #
  printf "`date` Save partition table information\n" | tee -a $LOGFILE
  parted $cmsdev unit s print >> $LOGFILE
  printf "`date` Delete extended paritoin\n" | tee -a $LOGFILE
  fdisk -c -u $cmsdev > /tmp/cms_del_part.log 2>&1 << !EOF
d
4
w
q
!EOF
  rtn=$?
  if [ $rtn -ne 0 ]; then
    grep "Device or resource busy" /tmp/cms_del_part.log > /dev/null
    if [ $? -ne 0 ]; then
      printf "Delete extended partion failed: $rtn\n" | tee -a $LOGFILE
      cat /tmp/cms_del_part.log | tee -a $LOGFILE
      rm -f /tmp/cms_del_part.log
      err_exit 1
    fi
  else
    cat /tmp/cms_del_part.log >> $LOGFILE
    rm -f /tmp/cms_del_part.log
  fi

  printf "`date` Repartition the disk\n" | tee -a $LOGFILE
  part_nums1=`cat /root/parted_new_size.sh | grep -v "^#" | wc -l`
  /root/parted_new_size.sh 2>&1 | tee /tmp/cms_part.log
  part_nums2=`grep "Device or resource busy" /tmp/cms_part.log | wc -l`
  cat /tmp/cms_part.log >> $LOGFILE
  rm -f /tmp/cms_part.log
  if [ $part_nums1 -ne $part_nums2 ]; then
    printf "`date` Repartition the disk failed\n" | tee -a $LOGFILE
    # restore to the old partitions
    printf "`date` Restore to the previous disk partitions\n" | tee -a $LOGFILE
    /root/parted_old_size.sh > /dev/null 2>&1
    rm -f /root/parted_old_size.sh
    printf "\nThe disk partition modification scripts are saved as\n" | tee -a $LOGFILE
    printf "`ls -rt /root/parted_*`\n\n" | tee -a $LOGFILE
    err_exit 1
  else
    printf "`date` Reboot the machine to make new partitions effective\n" >> $LOGFILE
    # make sure to reboot
    trap 'printf "`date` Reboot the machine\n"; reboot' 1 2 3 15
    rm -f /root/parted_old_size.sh
    printf "\nWARNING: A reboot is required before the new partitions become effective.\n"
    printf "After the reboot finishes, please re-run $0\n"
    printf "\nPress Enter to continue\n"
    read confirm
    printf "`date` Reboot the machine\n"
    reboot
    exit
  fi
}

# VMware Linux only
add_disks ()
{
  printf "`date` Adding disks to cmsdbs started\n" | tee -a $LOGFILE
  onstat - > /dev/null 2>&1
  if [ $? -ne 5 ]; then
    printf "\nInformix IDS must be On-Line before new disk(s) can be added\n\n" | tee -a $LOGFILE
    printf "`date` Adding new disk(s) to cmsdbs failed\n" | tee -a $LOGFILE
    exit 1
  fi
  # all other disks are for Informix
  # CMS-582: support adding disk for backup
  # remove raw devices that are not used by IDS (created by ids_Linux_setup)
  # to handle following scenario: 1) IDS turned on 2) add disks 3) reboot the machine
  # 4) mkfs on disk3 5) dbinit add_disks and it should skip disk3
  #
  for raw in `raw -qa | awk -F: '{print $1}' | egrep -v "/dev/raw/raw1$"`
  do
    onstat -d | grep $raw > /dev/null 2>&1
    if [ $? -ne 0 ]; then
      printf "`date` remove raw device mapping for $raw\n" | tee -a $LOGFILE
      raw $raw 0 0
      if [ $? -ne 0 ]; then
        printf "`date` remove raw device mapping for $raw failed\n" | tee -a $LOGFILE
        err_exit 1
      fi
    else
      printf "`date` $raw was already added to IDS\n" | tee -a $LOGFILE
    fi
  done
  j=1
  for disk in `sfdisk -s | grep -v ^total | sort | awk -F: '{print $1}'`
  do
    # skip boot disk
    if [ $j -eq 1 ]; then
      j=`expr $j + 1`
      continue
    fi
    # /sys/block/sda/device/vendor has vendor information
    grep -iq VMware /sys/block/${disk##*/}/device/vendor
    if [ $? -ne 0 ]; then
      printf "`date` $disk is not a VMware Virtual disk. Skip it.\n" | tee -a $LOGFILE
      continue
    else
      # Skip disks with file system. A disk may be added on a different
      # datastore for backup
      blkid | grep ${disk} > /dev/null
      if [ $? -eq 0 ]; then
        printf "`date` $disk has filesystem on it. Skip it.\n" | tee -a $LOGFILE
        continue
      fi
    fi
    dsize=`sfdisk -s $disk`
    printf "data disk $disk size: ${dsize}KB\n" >> $LOGFILE
    # align partition with 1MB boundary
    offset=1024
    dsize=$dsize-$offset
    dsize=$dsize/${pagesz}*${pagesz}
    raw -q /dev/raw/raw${j} > /dev/null 2>&1
    if [ $? -ne 0 ]; then
      raw /dev/raw/raw${j} $disk >> $LOGFILE 2>&1
      if [ $? -ne 0 ]; then
        printf "`date` raw /dev/raw/raw${j} $infdev failed\n" | tee -a $LOGFILE
        err_exit 1
      fi
      sleep 1
    fi
    chown informix:informix /dev/raw/raw${j}
    chmod 660 /dev/raw/raw${j}
    # add to cmsdbs
    onstat -d 2>&1 | grep "/dev/raw/raw${j}" > /dev/null
    if [ $? -ne 0 ]; then
      printf "onspaces -a cmsdbs -p /dev/raw/raw${j} -o $offset -s $dsize\n" \
        | tee -a $LOGFILE
      # drop this chunk to remove disk
      printf "To remove disk $disk run: onspaces -d cmsdbs -p /dev/raw/raw${j} -o $offset\n" >> $LOGFILE
      onspaces -a cmsdbs -p /dev/raw/raw${j} -o $offset -s $dsize
      if [ $? -ne 0 ]; then
        printf "onspaces -a cmsdbs -p /dev/raw/raw${j} -o $offset -s $dsize failed\n" \
          | tee -a $LOGFILE
        err_exit 1
      else
        printf "new disk $disk was successfully added to cmsdbs\n"
      fi
    else
      printf "`date` $disk was already added to cmsdbs\n" | tee -a $LOGFILE
    fi
    j=`expr $j + 1`
  done
  printf "`date` New disk(s) were successfully added to cmsdbs\n" | tee -a $LOGFILE
}

# IDS default page size is 2K.
# IDS 11.50 Data pages per fragment Maximum Capacity per Table is 16,775,134
# For performance and capacity reason use 8K for CMS R16
integer pagesz=8
# roodbs, physical and logical logs page size is 2K
integer logpgsz=2

integer sec dsize offset
integer rootdbs logdbs physdbs dbtemp cmsdbs

cd /

# make sure id is root
uid=`id | cut -d '=' -f 2 | cut -d '(' -f 1`
if [ $uid -ne 0 ]; then
  printf "You must login as root to run $0\n" | tee -a $LOGFILE
  err_exit 1
fi

if [ ! -f /opt/informix/bin/setenv ]; then
  printf "/opt/informix/bin/setenv does not exist\n\n" | tee -a $LOGFILE
  err_exit 1
fi
. /opt/informix/bin/setenv

if [ "$1" != "noprompt" -a "$1" != "add_disks" ]; then
  printf "WARNING: $0 will initialize CMS database.\n" | tee -a $LOGFILE
  printf "WARNING: All data will be lost!!!\n" | tee -a $LOGFILE
  printf "Do you want to continue? (y or n) : "
  read confirm
  if [[ $confirm != "y" && $confirm != "Y" ]]; then
    printf "Do not continue!\n" >> $LOGFILE
    err_exit 1
  fi
fi

dmidecode -s system-manufacturer 2>/dev/null | grep VMware > /dev/null
if [ $? -eq 0 ]; then
  VMware=true
fi

if [ "$1" = "add_disks" ]; then
  if [ "$VMware" = "true" ]; then
    add_disks
  else
    printf "\nDisks can only be added to virtual CMS systems\n\n" | tee -a $LOGFILE
  fi
  exit 0
fi

printf "`date` Creating CMS database started\n" | tee -a $LOGFILE

# create symbolic link for IDS root path
if [ "`uname`" = SunOS ]; then
  printf "disk\n0\nquit\n\n" > /tmp/fmtcmd.$$
  disk=`format -f /tmp/fmtcmd.$$ 2>&1 | grep selecting | awk '{print $2}'`
  if [ -z $disk ]; then
      printf "Can't find boot disk through format command\n" | tee -a $LOGFILE
      rm -f /tmp/fmtcmd.$$
      err_exit 1
  fi
  rm -f /tmp/fmtcmd.$$
  rawdisk=/dev/rdsk/${disk}s7
  datadisk=/dev/rdsk/cmsdisk

  rm -f /dev/cmstape
  ln -s /dev/rmt/0c /dev/cmstape
  cmstape="\/dev\/cmstape"
else
  cmsdev=`sfdisk -s | grep -v ^total | sort | head -1 | awk -F: '{print $1}'`
  infdev=`fdisk -l | grep $cmsdev | tail -1 | awk '{print $1}'`
  printf "`date` disk partition for CMS informix: $infdev\n" | tee -a $LOGFILE

  # if necessary repartition the disk for VMware CMS
  if [ "$VMware" = "true" ]; then
    printf "`date` Repartition the disk for VMware CMS\n" | tee -a $LOGFILE
    repartition_disk
  fi

  raw -q /dev/raw/raw1 > /dev/null 2>&1
  if [ $? -ne 0 ]; then
    raw /dev/raw/raw1 $infdev >> $LOGFILE 2>&1
    if [ $? -ne 0 ]; then
      printf "`date` raw /dev/raw/raw1 $infdev failed\n" | tee -a $LOGFILE
      err_exit 1
    fi
    # If it is the first time run raw /dev/raw/raw1 $infedev and
    # chown informix:informix /dev/raw/raw1 after reboot the informix
    # owner can only last less than a second then changed to root
    # Workaround for this issue is to sleep 1 second before run chown command
    sleep 1
  fi
  rawdisk=/dev/raw/raw1
  datadisk=/cmsdisk

  rm -f /cmstape
  ln -s /dev/st0 /cmstape
  cmstape="\/cmstape"
fi

rm -f $datadisk
ln -s $rawdisk $datadisk

printf "`date` raw disk path: $rawdisk\n" | tee -a $LOGFILE

# test the disk
dd if=$datadisk of=/dev/null count=1 > /dev/null 2>&1
if [ $? -ne 0 ]; then
  printf "Can't read data disk: $datadisk\n" | tee -a $LOGFILE
  err_exit 1
fi

chown informix:informix $rawdisk
chmod 660 $rawdisk

# On Linux it can cause connection problem without cleaning up socket files in /INFORMIXTMP
rm -rf /INFORMIXTMP

# Initialize IDS
printf "`date` Initializing Informix IDS started\n" | tee -a $LOGFILE
# make sure IDS is not running
ps -ef | grep oninit > /dev/null 2>&1
if [ $? -eq 0 ]; then
  onmode -ky > /dev/null 2>&1
  sleep 5
fi
# make sure no shared memory segments left
if [ "`uname`" = SunOS ]; then
  ipcs -m | grep ^m | grep informix > /dev/null
  if [ $? -eq 0 ]; then
    printf "`date` There are left over shared memory segments\n" >> $LOGFILE
    for i in `ipcs -m | grep ^m | grep informix | awk '{print $2}'`
    do
      ipcrm -m $i
    done
  fi
  ipcs -s | grep ^s | grep informix > /dev/null
  if [ $? -eq 0 ]; then
    printf "`date` There are left over semaphores\n" >> $LOGFILE
    for i in `ipcs -s | grep ^s | grep informix | awk '{print $2}'`
    do
      ipcrm -s $i
    done
  fi
else
  # On Linux some Informix shared memory segments are owned by root
  # Remove all shared memory segments and semophores. Since CMS is the only
  # application other than Informix uses shared memory and semophores there is
  # a small chance to mess up other applications.

  ipcs -m | grep informix > /dev/null
  if [ $? -eq 0 ]; then
    printf "`date` There are left over shared memory segments\n" >> $LOGFILE
    # remove Shared memory
    for i in `ipcs -m | egrep -v "\-|key|^$" | awk '{print $2}'`
    do
      ipcrm -m $i
    done
  fi

  ipcs -s | grep informix > /dev/null
  if [ $? -eq 0 ]; then
    printf "`date` There are left over semaphores\n" >> $LOGFILE
    # remove Semaphore
    for i in `ipcs -s | egrep -v "\-|key|^$" | awk '{print $2}'`
    do
      ipcrm -s $i
    done
  fi
fi

# create sqlhosts
if [ -f $INFORMIXDIR/etc/sqlhosts ]; then
  cp -p $INFORMIXDIR/etc/sqlhosts $INFORMIXDIR/etc/sqlhosts_`date "+%Y_%m_%d_%T"`
fi

# oninit fails if DBSERVERNAME has ".", "-" characters
# Invalid character(s) in DBSERVERNAME or DBSERVERALIASES (cms_emu.dr.avaya.com).
shortname=`hostname | awk -F. '{print $1}' | sed /-/s//_/g`

if [ "`uname`" = SunOS ]; then
  printf "cms_ol onipcstr `uname -n` cms_ol\n" > $INFORMIXDIR/etc/sqlhosts
  printf "oacms_ol onipcstr `uname -n` oacms_ol\n" >> $INFORMIXDIR/etc/sqlhosts
  printf "cms_net ontlitcp `uname -n` 50000\n" >> $INFORMIXDIR/etc/sqlhosts
  printf "cms_${shortname} ontlitcp `uname -n` 50001\n" >> $INFORMIXDIR/etc/sqlhosts
else
  printf "cms_ol onipcstr `uname -n` cms_ol\n" > $INFORMIXDIR/etc/sqlhosts
  printf "oacms_ol onipcstr `uname -n` oacms_ol\n" >> $INFORMIXDIR/etc/sqlhosts
  printf "cms_net onsoctcp `uname -n` 50000\n" >> $INFORMIXDIR/etc/sqlhosts
  printf "cms_${shortname} onsoctcp `uname -n` 50001\n" >> $INFORMIXDIR/etc/sqlhosts
fi
chown informix:informix $INFORMIXDIR/etc/sqlhosts

# check console.msgs file
if [ ! -f $INFORMIXDIR/console.msgs ]; then
  touch $INFORMIXDIR/console.msgs
  chown informix:informix $INFORMIXDIR/console.msgs
fi

# setup sqlhosts for isql
cp -p $INFORMIXDIR/etc/sqlhosts /opt/informix_isql32/etc/sqlhosts

printf "`date` Checking kernel parameters\n" | tee -a $LOGFILE

integer shmem
if [ "`uname`" = SunOS ]; then
  # Kernel Parameters required by IDS
  # IDS Failure with a Large Intimate Shared Memory Page Size
  # set mmu_ism_pagesize = 4194304
  # Don't need to rebbot the machine since CMS installation requires reboot
  # 05/20/2011 mmu_ism_pagesize is no longer in kernel for Solaris 10 update 9
  # Comment out following 5 lines
  #val=`grep mmu_ism_pagesize /etc/system | awk -F'=' '{print $2}'`
  #if [ -z $val ]; then
  #    printf "add mmu_ism_pagesize=4194304\n" | tee -a $LOGFILE
  #    printf "set mmu_ism_pagesize=4194304\n" >> /etc/system
  #fi
  # Tune IPC for Solaris 10. See IDS release notes for details
  # /etc/system shmsys:shminfo_shmmax sets system project max-shm-memory to
  # a huge number. The root.user and default project max-shm-memory remain
  # the same as shmsys:shminfo_shmmax. If IDS is started by init it belongs
  # to system project otherwise it uses user.root project.
  shmem=`prtconf | grep "Memory size" | awk '{print $3}'`
  printf "`date` Max shared Memory size: $shmem MB\n" | tee -a $LOGFILE
  prctl -n project.max-shm-memory -r -v ${shmem}mb -i project system user.root noproject default 2>&1 | tee -a $LOGFILE
  # Note that the project.max-shm-memory resource control limits the total
  # amount of shared memory of one project, whereas previously,
  # the shmsys:shminfo_shmmax parameter limited the
  # size of a single shared memory segment
  # For CMS R16, use old /etc/system way to set shmmax
  # which is done by /cms/install/bin/SETtunes
  #projmod -s -K "project.max-shm-memory=(priv,${shmem}mb,deny)"\
  #           user.root
  #projmod -A user.root
else
  # Linux kernel parameters
  # ids_machine_notes_11.50.txt: Informix tested kernel parameters as following
  # SHMMAX: 4398046511104 SHMMIN: 1 SHMMNI: 4096 SHMSEG: 128 SHMALL: 4194304
  # SEMMNI: 4096 SEMMSL: 250 SEMMNS: 32000 SEMOPM: 32
  # SEMMSL should be set to at least 100

  shmem=`grep MemTotal /proc/meminfo | awk '{print $2}'`
  printf "`date` Max shared Memory size: $shmem KB\n" | tee -a $LOGFILE
  shmem=$shmem*1024
  integer shmall=$shmem/16

  # Out of box Linux kernel parameters are good for Informix
  # Just tune semaphore parmaters
  sysctl -p - << !EOF | tee -a $LOGFILE

  # Controls the maximum shared segment size, in bytes
  #kernel.shmmax = $shmem

  # Controls the maximum number of shared memory segments, in pages
  #kernel.shmall = $shmall

  kernel.sem = 250 256000 32 4096
!EOF
fi

# Make sure Physical Log fit in rootdbs
# Physical Log might be increased bigger than rootdbs for performance reason
# reset TAPEDEV to /dev/null so that ontape -s can finish quickly
# for physical log modification
cp -p $INFORMIXDIR/etc/onconfig.cms $INFORMIXDIR/etc/onconfig.cms.sv
cp -p $INFORMIXDIR/etc/onconfig.cms $INFORMIXDIR/etc/onconfig.cms.old
sed -e "/^PHYSFILE/s/PHYSFILE.*/PHYSFILE 50000/" \
    -e "/^TAPEDEV/s/TAPEDEV.*/TAPEDEV \/dev\/null/" \
    -e "/^TAPEBLK/s/TAPEBLK.*/TAPEBLK 4096/" \
    -e "/^LTAPEBLK/s/LTAPEBLK.*/LTAPEBLK 4096/" \
    -e "/^IFX_FOLDVIEW/s/IFX_FOLDVIEW.*/IFX_FOLDVIEW 1/" \
    -e "/^FULL_DISK_INIT/s/.*/FULL_DISK_INIT 1/" \
    -e "/^DBSERVERALIASES/s/.*/DBSERVERALIASES oacms_ol,cms_net,cms_${shortname}/" \
    $INFORMIXDIR/etc/onconfig.cms.old > $INFORMIXDIR/etc/onconfig.cms
oninit -ivy
if [ $? -ne 0 ]; then
  printf "`date` Initializing Informix failed\n" | tee -a $LOGFILE
  err_exit 1
fi
# give ids a little time to initialize before checking if it is online
integer cnt=0
  while true
  do
    onstat -m | grep sysadmin | grep successfully > /dev/null
    if [ $? -ne 0 ]; then
      sleep 5
      cnt=$cnt+1
    else
      break
    fi
    # Timeout
    if [ $cnt -eq 30 ]; then
      printf "`date` IDS did not initialize properly\n" | tee -a $LOGFILE
      err_exit 1
    fi
done

printf "`date` Initializing Informix IDS successfully finished\n" | tee -a $LOGFILE

printf "`date` Creating CMS dbspaces started\n" | tee -a $LOGFILE
#
# For Solaris:
# Creating dbspaces on boot disk s7
# onstat -d shows size in pages
# onspaces using KB as unit
#
# Partition       Size           Name
# 0               22GB           root          /
# 1               8GB            swap
# 2               full_disk      backup
# 3               10GB           unassigned    /cms
# 4               26GB           var           /var
# 5               200GB          unassigned    /storage
# 6               32GB           home          /home
# 7               rest_of_disk   unassigned    Informix
# 8 (x86)         1 cyl (~16MB)  boot          boot loader (start from 0)
#
# For Linux:
# Creating dbspaces on boot disk s11
#
# Partition       Size           System ID
# 1               600MB          Linux(0x83)   /boot
# 2               10GB           Linux(0x83)   /
# 3               10GB           Linux(0x83)   /cms
# 4               total-1-2-3    Linux(0x0f)   extended
# 5               8GB            swap (0x82)   swap
# 6               190000MB       Linux(0x83)   /storage
# 7               32GB           Linux(0x83)   /export/home
# 8               26624MB        Linux(0x83)   /var
# 9               16GB           Linux(0x83)   /tmp
# 10              12224MB        Linux(0x83)   /opt
# 11              rest_of_disk   Linux(0x83)   Informix

if [ "`uname`" = SunOS ]; then
  # get number of disk sectors on boot disk s7
  sec=`prtvtoc $datadisk | grep "     7     " | awk '{print $5}'`
  # $sec*512: bytes
  # disk size in KB
  dsize=$sec/2
else
  # For Linux use sfdisk
  # Linux uses 1K block from sfdisk
  dsize=`sfdisk -s $infdev`
fi
printf "data disk $infdev size: ${dsize}KB\n" >> $LOGFILE

# rootdbs size is defined in $INFORMIXDIR/etc/onconfig.cms
# 256000 KB for CMS R16
rootdbs=`grep ^ROOTSIZE $INFORMIXDIR/etc/onconfig.cms | awk '{print $2}'`
offset=$rootdbs
# 640M physdbs
# log files cannot be created on dbspaces of big page
# phydbs page size is still 2K
physdbs=655360
onspaces -c -d physdbs -p $datadisk -o $offset -s $physdbs
if [ $? -ne 0 ]; then
  printf "onspaces -c -d physdbs -p $datadisk -o $offset -s $physdbs failed\n" \
        | tee -a $LOGFILE
  err_exit 1
fi

# 128MB logdbs
logdbs=131072
offset=$offset+$physdbs
onspaces -c -d logdbs -p $datadisk -o $offset -s $logdbs
if [ $? -ne 0 ]; then
  printf "onspaces -c -d logdbs -p $datadisk -o $offset -s $logdbs failed\n" \
        | tee -a $LOGFILE
  err_exit 1
fi

# 5GB dbtemp (need to test)
# for small testing/demo system (less than 25GB data disk) 15% for dbtemp
if [ $dsize -lt 26214400 ]; then
  dbtemp=$dsize/100*15/${pagesz}*${pagesz}
else
  dbtemp=5242880
fi
offset=$offset+$logdbs
onspaces -c -d dbtemp -k $pagesz -p $datadisk -o $offset -s $dbtemp
if [ $? -ne 0 ]; then
  printf "onspaces -c -d dbtemp -k $pagesz -p $datadisk -o $offset -s $dbtemp failed\n" \
        | tee -a $LOGFILE
  err_exit 1
fi

# The rest on s7 assign to cmsdbs
offset=$offset+$dbtemp
dsize=$dsize-$offset
# wi00939992. Leave 1GB unused
# This is for binary db backup. For small/demo system no need to waste 1GB
if [ $dsize -gt 26214400 ]; then
  dsize=$dsize-1048576
fi
# chun size must be multiple of page size (8K)
# i=19/8*8 then i=16
dsize=$dsize/${pagesz}*${pagesz}
onspaces -c -d cmsdbs -k $pagesz -p $datadisk -o $offset -s $dsize
if [ $? -ne 0 ]; then
  printf "onspaces -c -d cmsdbs -p $datadisk -o $offset -s $dsize failed\n" \
        | tee -a $LOGFILE
  err_exit 1
fi

#
# Move physical and logical logs to physdbs and logdbs
#

# Change physical log to physdbs
# Using onstat -d to find free space in physdbs
# Using onstat -l to display physical and logical logs information
physdbs=`onstat -d | grep "  2      2  " | awk '{print $6}'`
# convert from page to KB
physdbs=${physdbs}*${logpgsz}
onparams -p -s $physdbs -d physdbs -y
if [ $? -ne 0 ]; then
  printf "onparams -p -s $physdbs -d physdbs -y failed\n" | tee -a $LOGFILE
  err_exit 1
fi

# The physical log has been modified. Do a full archive
ontape -s >> $LOGFILE

# switch logical log to logdbs
# add logical logs to logdbs
# onparams will change onconfig.cms LOGFILES value to match
# number of logs added by onparams
# keep number of logs same as before so that no need to modify
# onconfig.cms when we have to run onini -i
integer i=0 logsize lognum=3
logdbs=`onstat -d | grep "  3      3  " | awk '{print $6}'`
logsize=${logdbs}/${lognum}*${logpgsz}
while (((i=$i+1)<$lognum))
do
  onparams -a -d logdbs -s $logsize
  if [ $? -ne 0 ]; then
    printf "onparams -a -d logdbs -s $logsize failed\n" | tee -a $LOGFILE
    err_exit 1
  fi
done
# Round up remainder pages to the last log file
# number of pages
lognum=$lognum-1
logsize=logdbs-logsize/2*${lognum}
# last log file in KB
logsize=logsize*2
onparams -a -d logdbs -s $logsize
if [ $? -ne 0 ]; then
  printf "onparams -a -d logdbs -s $logsize failed\n" | tee -a $LOGFILE
  err_exit 1
fi

#
# Drop logical logs in rootdbs
#

# Move current logical log to the next one
# Past the first 3 located in rootdbs
onmode -l; onmode -l; onmode -l
# Force a checkpoint that flushes buffers to disk
onmode -c
# Drop logs 1, 2, and 3
i=0
while (((i=$i+1)<4))
do
  onparams -d -l $i -y
  if [ $? -ne 0 ]; then
    printf "onparams -d -l $i -y failed\n" | tee -a $LOGFILE
    err_exit 1
  fi
done

# add disks for VMware CMS
if [ "$VMware" = "true" ]; then
  add_disks
fi

# Logical log files have been pre-dropped
# In order to delete them from the log list and reuse the space
# need a level 0 archives for all non-temp dbspaces
ontape -s >> $LOGFILE
printf "`date` Creating CMS dbspaces successfully finished\n" | tee -a $LOGFILE

printf "`date` Configuring tape drive to `printf $cmstape | sed '/\\\/s/\\\//g'`\n" \
        | tee -a $LOGFILE
# You can change the values of parameters for ontape while the database
# server is online. The change takes effect immediately
# Change the TAPEDEV parameter or the LTAPEDEV parameter to /dev/null,
# You must use the ON-Monitor utility to make this change
# while the database server is online
cp -p $INFORMIXDIR/etc/onconfig.cms $INFORMIXDIR/etc/onconfig.cms.old
sed -e "/^TAPEDEV/s/TAPEDEV.*/TAPEDEV $cmstape/" \
    $INFORMIXDIR/etc/onconfig.cms.old > $INFORMIXDIR/etc/onconfig.cms
rm -rf $INFORMIXDIR/etc/onconfig.cms.old

# IDS 11.50 by default schedules update statistics daily
# Since CMS already runs update statistics
# Disable automatic statistics updating
printf "`date` Disabling IDS Automatic Statistics Updating\n" | tee -a $LOGFILE
su - informix -c "
. /opt/informix/bin/setenv
dbaccess sysadmin << !EOF
update ph_task set tk_enable='f' where tk_name='Auto Update Statistics Evaluation';
!EOF" >> $LOGFILE 2>&1
if [ $? -ne 0 ]; then
  printf "Disabling IDS Automatic Statistics Updating failed\n" | tee -a $LOGFILE
  err_exit 1
fi
printf "`date` Disabling IDS Automatic Statistics Updating successfully finished\n"\
      | tee -a $LOGFILE

printf "`date` Creating CMS database successfully finished\n\n" | tee -a $LOGFILE
```