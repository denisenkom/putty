cmake_minimum_required(VERSION 3.2)
project(putty)

#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

find_package(GTK2 2.6)

IF( WIN32 )
    add_definitions(-D_WINDOWS)
    SET ( CMAKE_EXE_LINKER_FLAGS /MANIFEST:NO )
ENDIF( WIN32 )

if(APPLE)
    add_definitions(-DMACOSX)
endif(APPLE)

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
 sshaes.c sshccp.c sshsh256.c sshsh512.c sshbn.c wildcard.c pinger.c ssharcf.c
 sshgssc.c pgssapi.c sshshare.c sshecc.c)
set(WINSSH ${SSH} windows/winnoise.c windows/winpgntc windows/wingss.c)
set(UXSSH ${SSH} unix/uxnoise.c unix/uxagentc.c unix/uxgss.c unix/uxshare.c)

# SFTP implementation (pscp, psftp).
set(SFTP sftp.c int64.c logging.c)

# Miscellaneous objects appearing in all the network utilities (not
# Pageant or PuTTYgen).
set(MISC timing.c callback.c misc.c version.c settings.c tree234.c proxy.c conf.c)
set(WINMISC ${MISC} windows/winstore.c windows/winnet.c windows/winhandl.c cmdline.c windows/windefs.c windows/winmisc.c windows/winproxy.c
    windows/wintime.c)
set(UXMISC ${MISC} unix/uxstore.c unix/uxsel.c unix/uxnet.c unix/uxpeer.c cmdline.c
        unix/uxmisc.c unix/uxproxy.c time.c)
set(OSXMISC ${MISC} unix/uxstore.c unix/uxsel.c macosx/osxsel.m unix/uxnet.c unix/uxpeer.c
        unix/uxmisc.c unix/uxproxy.c time.c)

# import.c and dependencies, for PuTTYgen-like utilities that have to
# load foreign key files.
set(IMPORT import.c sshbcrypt.c sshblowf.c)

# Character set library, for use in pterm.
set(CHARSET charset/sbcsdat.c charset/slookup.c charset/sbcs.c charset/utf8.c
        charset/toucs.c charset/fromucs.c charset/xenc.c charset/mimeenc.c
        charset/macenc.c charset/localenc.c)

# Network backend sets. This also brings in the relevant attachment
# to proxy.c depending on whether we're crypto-avoidant or not.
set(BE_ALL be_all.c cproxy.c)
set(BE_NOSSH be_nossh.c nocproxy.c)
set(BE_SSH be_ssh.c cproxy.c)
set(BE_NONE be_none.c nocproxy.c)
# More backend sets, with the additional Windows serial-port module.
set(W_BE_ALL be_all_s.c windows/winser.c cproxy.c)
set(W_BE_NOSSH be_nos_s.c windows/winser.c nocproxy.c)
# And with the Unix serial-port module.
set(U_BE_ALL be_all_s.c unix/uxser.c cproxy.c)
set(U_BE_NOSSH be_nos_s.c unix/uxser.c nocproxy.c)

# Standard libraries.
set(LIBS advapi32 user32 gdi32 comctl32 comdlg32
 shell32 winmm imm32 winspool ole32 crypt32)

#add_executable(puttytel WIN32 ${GUITERM} ${NONSSH} ${W_BE_NOSSH} ${WINMISC} windows/puttytel.rc nogss.c)
#target_link_libraries(puttytel ${LIBS})

if (WIN32)

add_executable(pageant WIN32 ${PAGEANT_SRC})
target_link_libraries(pageant ${LIBS})

add_executable(putty WIN32 ${GUITERM} ${NONSSH} ${WINSSH} ${W_BE_ALL} ${WINMISC} windows/winx11.c windows/putty.rc)
target_link_libraries(putty ${LIBS})

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

elseif(UNIX)

    if (GTK2_FOUND)
    add_executable(putty ${GTKTERM} unix/uxmisc.c misc.c ldisc.c settings.c unix/uxsel.c ${U_BE_ALL} unix/uxstore.c
        unix/uxsignal.c ${CHARSET} unix/uxputty.c ${NONSSH} ${UXSSH} ${UXMISC} unix/ux_x11.c unix/xpmputty.c
        unix/xpmpucfg.c)

    add_executable(pageant unix/uxpgnt.c unix/uxagentc.c pageant.c sshrsa.c sshpubk.c sshdes.c sshbn.c sshmd5.c
        version.c tree234.c misc.c sshaes.c sshsha.c sshdss.c sshsh256.c sshsh512.c sshecc.c
        conf.c unix/uxsignal.c nocproxy.c nogss.c be_none.c x11fwd unix/ux_x11.c unix/uxcons.c unix/gtkask.c
        unix/gtkmisc.c ${UXMISC})
    endif()

add_executable(plink unix/uxplink.c unix/uxcons.c ${NONSSH} ${UXSSH} ${U_BE_ALL} logging.c ${UXMISC} unix/uxsignal.c
        unix/ux_x11.c noterm.c)

add_executable(puttygen cmdgen.c sshrsag.c sshdssg.c sshprime.c sshdes.c sshbn.c sshmd5.c version.c
    sshrand.c unix/uxnoise.c sshsha.c misc.c sshrsa.c sshdss.c unix/uxcons.c unix/uxstore.c unix/uxmisc.c
    sshpubk.c sshaes.c sshsh256.c sshsh512.c ${IMPORT} time.c tree234.c
    unix/uxgen.c notiming.c conf.c sshecc.c sshecdsag.c)

add_executable(pscp pscp.c unix/uxsftp.c unix/uxcons.c ${UXSSH} ${BE_SSH} ${SFTP} wildcard.c ${UXMISC})
add_executable(psftp psftp.c unix/uxsftp.c unix/uxcons.c ${UXSSH} ${BE_SSH} ${SFTP} wildcard.c ${UXMISC})

endif()

if(APPLE)
    #add_executable(PuTTY macosx/osxmain.m ${OSXTERM} ${OSXMISC} ${CHARSET} ${U_BE_ALL} ${NONSSH} ${UXSSH}
    #    unix/ux_x11.c unix/uxpty.c unix/uxsignal.c testback.c)
endif()
