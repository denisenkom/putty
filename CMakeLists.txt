cmake_minimum_required(VERSION 3.2)
project(putty)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

IF( WIN32 )
    add_definitions(-D_WINDOWS)
    SET ( CMAKE_EXE_LINKER_FLAGS /MANIFEST:NO )
ENDIF( WIN32 )

if(APPLE)
    add_definitions(-DMACOSX)
endif(APPLE)

add_definitions(-DSECURITY_WIN32)

include_directories(. windows macosx unix charset)

set(PAGEANT_SRC sshrsa.c sshpubk.c sshdes.c sshbn.c sshmd5.c version.c
               tree234.c misc.c sshaes.c sshsha.c sshdss.c sshsh256.c sshsh512.c
			   storage.h)

if (WIN32)
    set(PAGEANT_SRC ${PAGEANT_SRC} windows/winpgnt.c windows/winpgntc.c windows/winutils.c windows/winmisc.c
        windows/winhelp.c windows/pageant.rc windows/winstore.c windows/winstuff.h windows/winjump.c sc.c sc.h)
endif (WIN32)

if (UNIX)
    set(PAGEANT_SRC ${PAGEANT_SRC} unix/uxmisc.c unix/uxnoise.c unix/uxstore.c sshrand.c timing.c)
endif(UNIX)

if (APPLE)
    set(PAGEANT_SRC ${PAGEANT_SRC} macosx/osxmain.m macosx/osxdlg.m)
endif (APPLE)

# Terminal emulator and its (platform-independent) dependencies.
set(TERMINAL terminal.c wcwidth.c ldiscucs.c logging.c tree234.c minibidi.c
 config.c dialog.c)

# GUI front end and terminal emulator (putty, puttytel).
set(GUITERM ${TERMINAL} windows/window.c windows/windlg.c windows/winctrls.c windows/sizetip.c windows/winucs.c windows/winprint.c
 windows/winutils.c windows/wincfg.c sercfg.c windows/winhelp.c windows/winjump.c)

# Non-SSH back ends (putty, puttytel, plink).
set(NONSSH telnet.c raw.c rlogin.c ldisc.c pinger.c)

# SSH back end (putty, plink, pscp, psftp).
set(SSH ssh.c sshcrc.c sshdes.c sshmd5.c sshrsa.c sshrand.c sshsha.c sshblowf.c
 sshdh.c sshcrcda.c sshpubk.c sshzlib.c sshdss.c x11fwd.c portfwd.c
 sshaes.c sshsh256.c sshsh512.c sshbn.c wildcard.c pinger.c ssharcf.c
 sshgssc.c pgssapi.c)
set(WINSSH ${SSH} windows/winnoise.c windows/winpgntc windows/wingss.c)

# SFTP implementation (pscp, psftp).
set(SFTP sftp.c int64.c logging.c)

# Miscellaneous objects appearing in all the network utilities (not
# Pageant or PuTTYgen).
set(MISC timing.c misc.c version.c settings.c tree234.c proxy.c)
set(WINMISC ${MISC} windows/winstore.c windows/winnet.c windows/winhandl.c cmdline.c windows/windefs.c windows/winmisc.c windows/winproxy.c
    windows/wintime.c)

# Network backend sets. This also brings in the relevant attachment
# to proxy.c depending on whether we're crypto-avoidant or not.
set(BE_ALL be_all.c cproxy.c)
set(BE_NOSSH be_nossh.c nocproxy.c)
set(BE_SSH be_ssh.c cproxy.c)
set(BE_NONE be_none.c nocproxy.c)
# More backend sets, with the additional Windows serial-port module.
set(W_BE_ALL be_all_s.c windows/winser.c cproxy.c)
set(W_BE_NOSSH be_nos_s.c windows/winser.c nocproxy.c)

# Standard libraries.
set(LIBS advapi32 user32 gdi32 comctl32 comdlg32
 shell32 winmm imm32 winspool ole32 crypt32)

add_executable(pageant WIN32 ${PAGEANT_SRC})

target_link_libraries(pageant core mw-agent-improvement)

if (WIN32)
target_link_libraries(pageant ${LIBS})
endif (WIN32)

add_executable(putty WIN32 ${GUITERM} ${NONSSH} ${WINSSH} ${W_BE_ALL} ${WINMISC} windows/winx11.c windows/putty.rc)
target_link_libraries(putty ${LIBS})

#add_executable(puttytel WIN32 ${GUITERM} ${NONSSH} ${W_BE_NOSSH} ${WINMISC} windows/puttytel.rc nogss.c)
#target_link_libraries(puttytel ${LIBS})

add_executable(plink windows/winplink.c windows/wincons.c ${NONSSH} ${WINSSH} ${W_BE_ALL} logging.c ${WINMISC}
    windows/winx11.c windows/plink.rc windows/winnojmp.c)
target_link_libraries(plink ${LIBS})

add_executable(pscp pscp.c windows/winsftp.c windows/wincons.c ${WINSSH} ${BE_SSH} ${SFTP} wildcard.c ${WINMISC}
    windows/pscp.rc windows/winnojmp.c)
target_link_libraries(pscp ${LIBS})

add_executable(psftp psftp.c windows/winsftp.c windows/wincons.c ${WINSSH} ${BE_SSH} ${SFTP} wildcard.c ${WINMISC}
    windows/psftp.rc windows/winnojmp.c)
target_link_libraries(psftp ${LIBS})

add_executable(puttygen WIN32 windows/winpgen.c sshrsag.c sshdssg.c sshprime.c sshdes.c sshbn.c sshmd5.c version.c
    sshrand.c windows/winnoise.c sshsha.c windows/winstore.c misc.c windows/winctrls.c sshrsa.c sshdss.c windows/winmisc.c
    sshpubk.c sshaes.c sshsh256.c sshsh512.c import.c windows/winutils.c windows/puttygen.rc
    tree234.c notiming.c windows/winhelp.c windows/winnojmp.c windows/wintime.c)
target_link_libraries(puttygen ${LIBS})
