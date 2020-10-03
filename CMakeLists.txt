cmake_minimum_required(VERSION 3.4 FATAL_ERROR)

# Enforce "Debug" as standard build type.
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Debug" CACHE STRING "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif()

# CMake project configuration.
project(vkQuake2)
set(VKQ2_VERSION_MAJOR 1)
set(VKQ2_VERSION_MINOR 5)
set(VKQ2_VERSION_POINT 3)

# Add extended path for FreeBSD and Homebrew on OS X.
list(APPEND CMAKE_PREFIX_PATH /usr/local)

if(UNIX)
  include(GNUInstallDirs)
endif()

# Architecture string.
set(Q2ARCH "${CMAKE_SYSTEM_PROCESSOR}" CACHE STRING "Override CPU architecture")
string(REGEX REPLACE "amd64" "x86_64" KERNEL_ARCH "${Q2ARCH}")

if("${KERNEL_ARCH}" STREQUAL "x86_64")
  set(ARCH "x64")
else()
  message(FATAL_ERROR "Unsupported ARCH: ${KERNEL_ARCH}")
endif()

# Set directory locations (allowing us to move directories easily)
set(MOUNT_DIR ${CMAKE_CURRENT_LIST_DIR})
set(BUILD_DIR ${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}${ARCH})
set(CLIENT_DIR ${MOUNT_DIR}/client)
set(SERVER_DIR ${MOUNT_DIR}/server)
set(REF_SOFT_DIR ${MOUNT_DIR}/ref_soft)
set(REF_GL_DIR ${MOUNT_DIR}/ref_gl)
set(REF_VK_DIR ${MOUNT_DIR}/ref_vk)
set(COMMON_DIR ${MOUNT_DIR}/qcommon)
set(LINUX_DIR ${MOUNT_DIR}/linux)
set(GAME_DIR ${MOUNT_DIR}/game)
set(NULL_DIR ${MOUNT_DIR}/null)
set(CTF_DIR ${MOUNT_DIR}/ctf)
set(XATRIX_DIR ${MOUNT_DIR}/xatrix)
set(ROGUE_DIR ${MOUNT_DIR}/rogue)
set(ZAERO_DIR ${MOUNT_DIR}/zaero)

# We need an Vulkan implementation.
find_package(Vulkan REQUIRED)

# Configure compiler flags
if(UNIX)
  # Base flags
  set(BASE_CFLAGS "-Dstricmp=strcasecmp -D_GNU_SOURCE")
  set(BASE_CFLAGS "${BASE_CFLAGS} -Wno-format-truncation -Wno-unused-result")
  # Flags for release build
  set(RELEASE_CFLAGS "${BASE_CFLAGS} -DNDEBUG -O3 -ffast-math -funroll-loops")
  set(RELEASE_CFLAGS "${RELEASE_CFLAGS} -fomit-frame-pointer -fexpensive-optimizations")
  # Flags for debug build
  set(DEBUG_CFLAGS "${BASE_CFLAGS} -g") # -D_DEBUG
endif()

# Linker flags
if(UNIX)
  set(LDFLAGS "-L/usr/local/lib -ldl -lm -lasound -lpthread")
  set(SVGALDFLAGS "-lvga -lm")
  set(XLDFLAGS "-L/usr/X11R6/lib -L/usr/local/lib -lX11 -lXext -lXxf86dga")
  # OpenGL renderer
  set(GLXLDFLAGS "-L/usr/X11R6/lib -L/usr/local/lib")
  set(GLXLDFLAGS "${GLXLDFLAGS} -lX11 -lXext -lXxf86dga -lXxf86vm -lm")
  # Vulkan renderer
  set(VKLDFLAGS "-lX11 -lXext -lXxf86dga -lXxf86vm -lm -lstdc++")
  set(VKCFLAGS ${Vulkan_INCLUDE_DIR})
endif()

# Shared library
if(UNIX)
  set(SHLIBEXT ${CMAKE_SHARED_LIBRARY_SUFFIX})
  set(SHLIBCFLAGS "-fPIC -std=c99")
  set(SHLIBCPPFLAGS "-fPIC -std=c++11")
  set(SHLIBLDFLAGS "-shared")
endif()

# Set compiler flags
if(CMAKE_BUILD_TYPE MATCHES "Debug")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${DEBUG_CFLAGS}")
else()
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${RELEASE_CFLAGS}")
endif()

################################################################################
# CLIENT/SERVER
################################################################################

set(QUAKE2_SOURCES
  ${CLIENT_DIR}/cl_cin.c
  ${CLIENT_DIR}/cl_ents.c
  ${CLIENT_DIR}/cl_fx.c
  ${CLIENT_DIR}/cl_input.c
  ${CLIENT_DIR}/cl_inv.c
  ${CLIENT_DIR}/cl_main.c
  ${CLIENT_DIR}/cl_newfx.c
  ${CLIENT_DIR}/cl_parse.c
  ${CLIENT_DIR}/cl_pred.c
  ${CLIENT_DIR}/cl_tent.c
  ${CLIENT_DIR}/cl_scrn.c
  ${CLIENT_DIR}/cl_view.c
  ${CLIENT_DIR}/console.c
  ${CLIENT_DIR}/keys.c
  ${CLIENT_DIR}/menu.c
  ${CLIENT_DIR}/snd_dma.c
  ${CLIENT_DIR}/snd_mem.c
  ${CLIENT_DIR}/snd_miniaudio.c
  ${CLIENT_DIR}/snd_mix.c
  ${CLIENT_DIR}/qmenu.c
  ${COMMON_DIR}/cmd.c
  ${COMMON_DIR}/cmodel.c
  ${COMMON_DIR}/common.c
  ${COMMON_DIR}/crc.c
  ${COMMON_DIR}/cvar.c
  ${COMMON_DIR}/files.c
  ${COMMON_DIR}/md4.c
  ${COMMON_DIR}/net_chan.c
  ${COMMON_DIR}/pmove.c
  ${GAME_DIR}/m_flash.c
  ${GAME_DIR}/q_shared.c
  ${SERVER_DIR}/sv_ccmds.c
  ${SERVER_DIR}/sv_ents.c
  ${SERVER_DIR}/sv_game.c
  ${SERVER_DIR}/sv_init.c
  ${SERVER_DIR}/sv_main.c
  ${SERVER_DIR}/sv_send.c
  ${SERVER_DIR}/sv_user.c
  ${SERVER_DIR}/sv_world.c
)

if(UNIX)
  set(QUAKE2_SOURCES
    ${QUAKE2_SOURCES}
    ${LINUX_DIR}/cd_linux.c
    ${LINUX_DIR}/q_shlinux.c
    ${LINUX_DIR}/vid_menu.c
    ${LINUX_DIR}/vid_so.c
    ${LINUX_DIR}/snd_alsa.c
    ${LINUX_DIR}/snd_mixa.s
    ${LINUX_DIR}/sys_linux.c
    ${LINUX_DIR}/glob.c
    ${LINUX_DIR}/net_udp.c
  )
endif()

set(QUAKE2_HEADERS
  ${CLIENT_DIR}/adivtab.h
  ${CLIENT_DIR}/anorms.h
  ${CLIENT_DIR}/asm_i386.h
  ${CLIENT_DIR}/block16.h
  ${CLIENT_DIR}/block8.h
  ${CLIENT_DIR}/cdaudio.h
  ${CLIENT_DIR}/client.h
  ${CLIENT_DIR}/console.h
  ${CLIENT_DIR}/input.h
  ${CLIENT_DIR}/keys.h
  ${CLIENT_DIR}/miniaudio.h
  ${CLIENT_DIR}/qmenu.h
  ${CLIENT_DIR}/ref.h
  ${CLIENT_DIR}/screen.h
  ${CLIENT_DIR}/snd_loc.h
  ${CLIENT_DIR}/snd_miniaudio.h
  ${CLIENT_DIR}/sound.h
  ${CLIENT_DIR}/vid.h
)

################################################################################
# DEDICATED SERVER
################################################################################

set(Q2DED_SOURCES
  ${COMMON_DIR}/cmd.c
  ${COMMON_DIR}/cmodel.c
  ${COMMON_DIR}/common.c
  ${COMMON_DIR}/crc.c
  ${COMMON_DIR}/cvar.c
  ${COMMON_DIR}/files.c
  ${COMMON_DIR}/md4.c
  ${COMMON_DIR}/net_chan.c
  ${COMMON_DIR}/pmove.c
  ${GAME_DIR}/q_shared.c
  ${SERVER_DIR}/sv_ccmds.c
  ${SERVER_DIR}/sv_ents.c
  ${SERVER_DIR}/sv_game.c
  ${SERVER_DIR}/sv_init.c
  ${SERVER_DIR}/sv_main.c
  ${SERVER_DIR}/sv_send.c
  ${SERVER_DIR}/sv_user.c
  ${SERVER_DIR}/sv_world.c
  ${NULL_DIR}/cd_null.c
  ${NULL_DIR}/cl_null.c
)

set(Q2DED_HEADERS
  ${COMMON_DIR}/crc.h
  ${COMMON_DIR}/qcommon.h
  ${COMMON_DIR}/qfiles.h
  ${GAME_DIR}/q_shared.h
  ${SERVER_DIR}/server.h
)

if(UNIX)
  set(Q2DED_SOURCES
    ${Q2DED_SOURCES}
    ${LINUX_DIR}/q_shlinux.c
    ${LINUX_DIR}/sys_linux.c
    ${LINUX_DIR}/glob.c
    ${LINUX_DIR}/net_udp.c
  )
endif()

################################################################################
# GAME
################################################################################

set(GAME_SOURCES
  ${GAME_DIR}/q_shared.c
  ${GAME_DIR}/g_ai.c
  ${GAME_DIR}/p_client.c
  ${GAME_DIR}/g_cmds.c
  ${GAME_DIR}/g_svcmds.c
  ${GAME_DIR}/g_chase.c
  ${GAME_DIR}/g_combat.c
  ${GAME_DIR}/g_func.c
  ${GAME_DIR}/g_items.c
  ${GAME_DIR}/g_main.c
  ${GAME_DIR}/g_misc.c
  ${GAME_DIR}/g_monster.c
  ${GAME_DIR}/g_phys.c
  ${GAME_DIR}/g_save.c
  ${GAME_DIR}/g_spawn.c
  ${GAME_DIR}/g_target.c
  ${GAME_DIR}/g_trigger.c
  ${GAME_DIR}/g_turret.c
  ${GAME_DIR}/g_utils.c
  ${GAME_DIR}/g_weapon.c
  ${GAME_DIR}/m_actor.c
  ${GAME_DIR}/m_berserk.c
  ${GAME_DIR}/m_boss2.c
  ${GAME_DIR}/m_boss3.c
  ${GAME_DIR}/m_boss31.c
  ${GAME_DIR}/m_boss32.c
  ${GAME_DIR}/m_brain.c
  ${GAME_DIR}/m_chick.c
  ${GAME_DIR}/m_flipper.c
  ${GAME_DIR}/m_float.c
  ${GAME_DIR}/m_flyer.c
  ${GAME_DIR}/m_gladiator.c
  ${GAME_DIR}/m_gunner.c
  ${GAME_DIR}/m_hover.c
  ${GAME_DIR}/m_infantry.c
  ${GAME_DIR}/m_insane.c
  ${GAME_DIR}/m_medic.c
  ${GAME_DIR}/m_move.c
  ${GAME_DIR}/m_mutant.c
  ${GAME_DIR}/m_parasite.c
  ${GAME_DIR}/m_soldier.c
  ${GAME_DIR}/m_supertank.c
  ${GAME_DIR}/m_tank.c
  ${GAME_DIR}/p_hud.c
  ${GAME_DIR}/p_trail.c
  ${GAME_DIR}/p_view.c
  ${GAME_DIR}/p_weapon.c
  ${GAME_DIR}/m_flash.c
)

set(GAME_HEADERS
  ${GAME_DIR}/game.h
  ${GAME_DIR}/g_local.h
  ${GAME_DIR}/m_actor.h
  ${GAME_DIR}/m_berserk.h
  ${GAME_DIR}/m_boss2.h
  ${GAME_DIR}/m_boss31.h
  ${GAME_DIR}/m_boss32.h
  ${GAME_DIR}/m_brain.h
  ${GAME_DIR}/m_chick.h
  ${GAME_DIR}/m_flipper.h
  ${GAME_DIR}/m_float.h
  ${GAME_DIR}/m_flyer.h
  ${GAME_DIR}/m_gladiator.h
  ${GAME_DIR}/m_gunner.h
  ${GAME_DIR}/m_hover.h
  ${GAME_DIR}/m_infantry.h
  ${GAME_DIR}/m_insane.h
  ${GAME_DIR}/m_medic.h
  ${GAME_DIR}/m_mutant.h
  ${GAME_DIR}/m_parasite.h
  ${GAME_DIR}/m_player.h
  ${GAME_DIR}/m_rider.h
  ${GAME_DIR}/m_soldier.h
  ${GAME_DIR}/m_supertank.h
  ${GAME_DIR}/m_tank.h
  ${GAME_DIR}/q_shared.h
)

################################################################################
# CTF
################################################################################

set(CTF_SOURCES
  ${CTF_DIR}/g_ai.c
  ${CTF_DIR}/g_chase.c
  ${CTF_DIR}/g_cmds.c
  ${CTF_DIR}/g_combat.c
  ${CTF_DIR}/g_ctf.c
  ${CTF_DIR}/g_func.c
  ${CTF_DIR}/g_items.c
  ${CTF_DIR}/g_main.c
  ${CTF_DIR}/g_misc.c
  ${CTF_DIR}/g_monster.c
  ${CTF_DIR}/g_phys.c
  ${CTF_DIR}/g_save.c
  ${CTF_DIR}/g_spawn.c
  ${CTF_DIR}/g_svcmds.c
  ${CTF_DIR}/g_target.c
  ${CTF_DIR}/g_trigger.c
  ${CTF_DIR}/g_utils.c
  ${CTF_DIR}/g_weapon.c
  ${CTF_DIR}/m_move.c
  ${CTF_DIR}/p_client.c
  ${CTF_DIR}/p_hud.c
  ${CTF_DIR}/p_menu.c
  ${CTF_DIR}/p_trail.c
  ${CTF_DIR}/p_view.c
  ${CTF_DIR}/p_weapon.c
  ${CTF_DIR}/q_shared.c
)

set(CTF_HEADERS
  ${CTF_DIR}/game.h
  ${CTF_DIR}/g_ctf.h
  ${CTF_DIR}/g_local.h
  ${CTF_DIR}/m_player.h
  ${CTF_DIR}/p_menu.h
  ${CTF_DIR}/q_shared.h
)

################################################################################
# XATRIX
################################################################################

set(XATRIX_SOURCES
  ${XATRIX_DIR}/g_ai.c
  ${XATRIX_DIR}/g_cmds.c
  ${XATRIX_DIR}/g_chase.c
  ${XATRIX_DIR}/g_combat.c
  ${XATRIX_DIR}/g_func.c
  ${XATRIX_DIR}/g_items.c
  ${XATRIX_DIR}/g_main.c
  ${XATRIX_DIR}/g_misc.c
  ${XATRIX_DIR}/g_monster.c
  ${XATRIX_DIR}/g_phys.c
  ${XATRIX_DIR}/g_save.c
  ${XATRIX_DIR}/g_spawn.c
  ${XATRIX_DIR}/g_svcmds.c
  ${XATRIX_DIR}/g_target.c
  ${XATRIX_DIR}/g_trigger.c
  ${XATRIX_DIR}/g_turret.c
  ${XATRIX_DIR}/g_utils.c
  ${XATRIX_DIR}/g_weapon.c
  ${XATRIX_DIR}/m_actor.c
  ${XATRIX_DIR}/m_berserk.c
  ${XATRIX_DIR}/m_boss2.c
  ${XATRIX_DIR}/m_boss3.c
  ${XATRIX_DIR}/m_boss31.c
  ${XATRIX_DIR}/m_boss32.c
  ${XATRIX_DIR}/m_boss5.c
  ${XATRIX_DIR}/m_brain.c
  ${XATRIX_DIR}/m_chick.c
  ${XATRIX_DIR}/m_fixbot.c
  ${XATRIX_DIR}/m_flash.c
  ${XATRIX_DIR}/m_flipper.c
  ${XATRIX_DIR}/m_float.c
  ${XATRIX_DIR}/m_flyer.c
  ${XATRIX_DIR}/m_gekk.c
  ${XATRIX_DIR}/m_gladb.c
  ${XATRIX_DIR}/m_gladiator.c
  ${XATRIX_DIR}/m_gunner.c
  ${XATRIX_DIR}/m_hover.c
  ${XATRIX_DIR}/m_infantry.c
  ${XATRIX_DIR}/m_insane.c
  ${XATRIX_DIR}/m_medic.c
  ${XATRIX_DIR}/m_move.c
  ${XATRIX_DIR}/m_mutant.c
  ${XATRIX_DIR}/m_parasite.c
  ${XATRIX_DIR}/m_soldier.c
  ${XATRIX_DIR}/m_supertank.c
  ${XATRIX_DIR}/m_tank.c
  ${XATRIX_DIR}/p_client.c
  ${XATRIX_DIR}/p_hud.c
  ${XATRIX_DIR}/p_trail.c
  ${XATRIX_DIR}/p_view.c
  ${XATRIX_DIR}/p_weapon.c
  ${XATRIX_DIR}/q_shared.c
)

set(XATRIX_HEADERS
  ${XATRIX_DIR}/game.h
  ${XATRIX_DIR}/g_local.h
  ${XATRIX_DIR}/m_actor.h
  ${XATRIX_DIR}/m_berserk.h
  ${XATRIX_DIR}/m_boss2.h
  ${XATRIX_DIR}/m_boss31.h
  ${XATRIX_DIR}/m_boss32.h
  ${XATRIX_DIR}/m_brain.h
  ${XATRIX_DIR}/m_chick.h
  ${XATRIX_DIR}/m_fixbot.h
  ${XATRIX_DIR}/m_flipper.h
  ${XATRIX_DIR}/m_float.h
  ${XATRIX_DIR}/m_flyer.h
  ${XATRIX_DIR}/m_gekk.h
  ${XATRIX_DIR}/m_gladiator.h
  ${XATRIX_DIR}/m_gunner.h
  ${XATRIX_DIR}/m_hover.h
  ${XATRIX_DIR}/m_infantry.h
  ${XATRIX_DIR}/m_insane.h
  ${XATRIX_DIR}/m_medic.h
  ${XATRIX_DIR}/m_mutant.h
  ${XATRIX_DIR}/m_parasite.h
  ${XATRIX_DIR}/m_player.h
  ${XATRIX_DIR}/m_rider.h
  ${XATRIX_DIR}/m_soldier.h
  ${XATRIX_DIR}/m_soldierh.h
  ${XATRIX_DIR}/m_supertank.h
  ${XATRIX_DIR}/m_tank.h
  ${XATRIX_DIR}/q_shared.h
)

################################################################################
# ROGUE
################################################################################

set(ROGUE_SOURCES
  ${ROGUE_DIR}/dm_ball.c
  ${ROGUE_DIR}/dm_tag.c
  ${ROGUE_DIR}/g_ai.c
  ${ROGUE_DIR}/g_chase.c
  ${ROGUE_DIR}/g_cmds.c
  ${ROGUE_DIR}/g_combat.c
  ${ROGUE_DIR}/g_func.c
  ${ROGUE_DIR}/g_items.c
  ${ROGUE_DIR}/g_main.c
  ${ROGUE_DIR}/g_misc.c
  ${ROGUE_DIR}/g_monster.c
  ${ROGUE_DIR}/g_newai.c
  ${ROGUE_DIR}/g_newdm.c
  ${ROGUE_DIR}/g_newfnc.c
  ${ROGUE_DIR}/g_newtarg.c
  ${ROGUE_DIR}/g_newtrig.c
  ${ROGUE_DIR}/g_newweap.c
  ${ROGUE_DIR}/g_phys.c
  ${ROGUE_DIR}/g_save.c
  ${ROGUE_DIR}/g_spawn.c
  ${ROGUE_DIR}/g_sphere.c
  ${ROGUE_DIR}/g_svcmds.c
  ${ROGUE_DIR}/g_target.c
  ${ROGUE_DIR}/g_trigger.c
  ${ROGUE_DIR}/g_turret.c
  ${ROGUE_DIR}/g_utils.c
  ${ROGUE_DIR}/g_weapon.c
  ${ROGUE_DIR}/m_actor.c
  ${ROGUE_DIR}/m_berserk.c
  ${ROGUE_DIR}/m_boss2.c
  ${ROGUE_DIR}/m_boss3.c
  ${ROGUE_DIR}/m_boss31.c
  ${ROGUE_DIR}/m_boss32.c
  ${ROGUE_DIR}/m_brain.c
  ${ROGUE_DIR}/m_carrier.c
  ${ROGUE_DIR}/m_chick.c
  ${ROGUE_DIR}/m_flash.c
  ${ROGUE_DIR}/m_flipper.c
  ${ROGUE_DIR}/m_float.c
  ${ROGUE_DIR}/m_flyer.c
  ${ROGUE_DIR}/m_gladiator.c
  ${ROGUE_DIR}/m_gunner.c
  ${ROGUE_DIR}/m_hover.c
  ${ROGUE_DIR}/m_infantry.c
  ${ROGUE_DIR}/m_insane.c
  ${ROGUE_DIR}/m_medic.c
  ${ROGUE_DIR}/m_move.c
  ${ROGUE_DIR}/m_mutant.c
  ${ROGUE_DIR}/m_parasite.c
  ${ROGUE_DIR}/m_soldier.c
  ${ROGUE_DIR}/m_stalker.c
  ${ROGUE_DIR}/m_supertank.c
  ${ROGUE_DIR}/m_tank.c
  ${ROGUE_DIR}/m_turret.c
  ${ROGUE_DIR}/m_widow.c
  ${ROGUE_DIR}/m_widow2.c
  ${ROGUE_DIR}/p_client.c
  ${ROGUE_DIR}/p_hud.c
  ${ROGUE_DIR}/p_trail.c
  ${ROGUE_DIR}/p_view.c
  ${ROGUE_DIR}/p_weapon.c
  ${ROGUE_DIR}/q_shared.c
)

set(ROGUE_HEADERS
  ${ROGUE_DIR}/game.h
  ${ROGUE_DIR}/g_local.h
  ${ROGUE_DIR}/m_actor.h
  ${ROGUE_DIR}/m_berserk.h
  ${ROGUE_DIR}/m_boss2.h
  ${ROGUE_DIR}/m_boss31.h
  ${ROGUE_DIR}/m_boss32.h
  ${ROGUE_DIR}/m_brain.h
  ${ROGUE_DIR}/m_carrier.h
  ${ROGUE_DIR}/m_chick.h
  ${ROGUE_DIR}/m_flipper.h
  ${ROGUE_DIR}/m_float.h
  ${ROGUE_DIR}/m_flyer.h
  ${ROGUE_DIR}/m_gladiator.h
  ${ROGUE_DIR}/m_gunner.h
  ${ROGUE_DIR}/m_hover.h
  ${ROGUE_DIR}/m_infantry.h
  ${ROGUE_DIR}/m_insane.h
  ${ROGUE_DIR}/m_medic.h
  ${ROGUE_DIR}/m_mutant.h
  ${ROGUE_DIR}/m_parasite.h
  ${ROGUE_DIR}/m_player.h
  ${ROGUE_DIR}/m_rider.h
  ${ROGUE_DIR}/m_soldier.h
  ${ROGUE_DIR}/m_stalker.h
  ${ROGUE_DIR}/m_supertank.h
  ${ROGUE_DIR}/m_tank.h
  ${ROGUE_DIR}/m_turret.h
  ${ROGUE_DIR}/m_widow2.h
  ${ROGUE_DIR}/m_widow.h
  ${ROGUE_DIR}/q_shared.h
)

################################################################################
# ZAERO
################################################################################

set(ZAERO_SOURCES
  ${ZAERO_DIR}/g_ai.c
  ${ZAERO_DIR}/g_cmds.c
  ${ZAERO_DIR}/g_combat.c
  ${ZAERO_DIR}/g_func.c
  ${ZAERO_DIR}/g_items.c
  ${ZAERO_DIR}/g_main.c
  ${ZAERO_DIR}/g_misc.c
  ${ZAERO_DIR}/g_monster.c
  ${ZAERO_DIR}/g_phys.c
  ${ZAERO_DIR}/g_save.c
  ${ZAERO_DIR}/g_spawn.c
  ${ZAERO_DIR}/g_svcmds.c
  ${ZAERO_DIR}/g_target.c
  ${ZAERO_DIR}/g_trigger.c
  ${ZAERO_DIR}/g_turret.c
  ${ZAERO_DIR}/g_utils.c
  ${ZAERO_DIR}/g_weapon.c
  ${ZAERO_DIR}/m_actor.c
  ${ZAERO_DIR}/m_berserk.c
  ${ZAERO_DIR}/m_boss2.c
  ${ZAERO_DIR}/m_boss3.c
  ${ZAERO_DIR}/m_boss31.c
  ${ZAERO_DIR}/m_boss32.c
  ${ZAERO_DIR}/m_brain.c
  ${ZAERO_DIR}/m_chick.c
  ${ZAERO_DIR}/m_flash.c
  ${ZAERO_DIR}/m_flipper.c
  ${ZAERO_DIR}/m_float.c
  ${ZAERO_DIR}/m_flyer.c
  ${ZAERO_DIR}/m_gladiator.c
  ${ZAERO_DIR}/m_gunner.c
  ${ZAERO_DIR}/m_hover.c
  ${ZAERO_DIR}/m_infantry.c
  ${ZAERO_DIR}/m_insane.c
  ${ZAERO_DIR}/m_medic.c
  ${ZAERO_DIR}/m_move.c
  ${ZAERO_DIR}/m_mutant.c
  ${ZAERO_DIR}/m_parasite.c
  ${ZAERO_DIR}/m_soldier.c
  ${ZAERO_DIR}/m_supertank.c
  ${ZAERO_DIR}/m_tank.c
  ${ZAERO_DIR}/p_client.c
  ${ZAERO_DIR}/p_hud.c
  ${ZAERO_DIR}/p_trail.c
  ${ZAERO_DIR}/p_view.c
  ${ZAERO_DIR}/p_weapon.c
  ${ZAERO_DIR}/q_shared.c
  ${ZAERO_DIR}/z_acannon.c
  ${ZAERO_DIR}/z_ai.c
  ${ZAERO_DIR}/z_anim.c
  ${ZAERO_DIR}/z_boss.c
  ${ZAERO_DIR}/z_camera.c
  ${ZAERO_DIR}/z_debug.c
  ${ZAERO_DIR}/z_frames.c
  ${ZAERO_DIR}/z_handler.c
  ${ZAERO_DIR}/z_hound.c
  ${ZAERO_DIR}/z_item.c
  ${ZAERO_DIR}/z_list.c
  ${ZAERO_DIR}/z_mtest.c
  ${ZAERO_DIR}/z_sentien.c
  ${ZAERO_DIR}/z_spawn.c
  ${ZAERO_DIR}/z_trigger.c
  ${ZAERO_DIR}/z_weapon.c
)

set(ZAERO_HEADERS
  ${ZAERO_DIR}/game.h
  ${ZAERO_DIR}/g_local.h
  ${ZAERO_DIR}/m_actor.h
  ${ZAERO_DIR}/m_berserk.h
  ${ZAERO_DIR}/m_boss2.h
  ${ZAERO_DIR}/m_boss31.h
  ${ZAERO_DIR}/m_boss32.h
  ${ZAERO_DIR}/m_brain.h
  ${ZAERO_DIR}/m_chick.h
  ${ZAERO_DIR}/m_flipper.h
  ${ZAERO_DIR}/m_float.h
  ${ZAERO_DIR}/m_flyer.h
  ${ZAERO_DIR}/m_gladiator.h
  ${ZAERO_DIR}/m_gunner.h
  ${ZAERO_DIR}/m_hover.h
  ${ZAERO_DIR}/m_infantry.h
  ${ZAERO_DIR}/m_insane.h
  ${ZAERO_DIR}/m_medic.h
  ${ZAERO_DIR}/m_mutant.h
  ${ZAERO_DIR}/m_parasite.h
  ${ZAERO_DIR}/m_player.h
  ${ZAERO_DIR}/m_rider.h
  ${ZAERO_DIR}/m_soldier.h
  ${ZAERO_DIR}/m_supertank.h
  ${ZAERO_DIR}/m_tank.h
  ${ZAERO_DIR}/q_shared.h
  ${ZAERO_DIR}/z_anim.h
  ${ZAERO_DIR}/z_boss.h
  ${ZAERO_DIR}/z_debug.h
  ${ZAERO_DIR}/z_frames.h
  ${ZAERO_DIR}/z_handler.h
  ${ZAERO_DIR}/z_hound.h
  ${ZAERO_DIR}/z_list.h
  ${ZAERO_DIR}/z_sentien.h
)

################################################################################
# REF_SOFT
################################################################################

# Software renderer
set(REF_SOFT_SOURCES
  ${REF_SOFT_DIR}/r_aclip.c
  ${REF_SOFT_DIR}/r_alias.c
  ${REF_SOFT_DIR}/r_bsp.c
  ${REF_SOFT_DIR}/r_draw.c
  ${REF_SOFT_DIR}/r_edge.c
  ${REF_SOFT_DIR}/r_image.c
  ${REF_SOFT_DIR}/r_light.c
  ${REF_SOFT_DIR}/r_main.c
  ${REF_SOFT_DIR}/r_misc.c
  ${REF_SOFT_DIR}/r_model.c
  ${REF_SOFT_DIR}/r_part.c
  ${REF_SOFT_DIR}/r_poly.c
  ${REF_SOFT_DIR}/r_polyse.c
  ${REF_SOFT_DIR}/r_rast.c
  ${REF_SOFT_DIR}/r_scan.c
  ${REF_SOFT_DIR}/r_sprite.c
  ${REF_SOFT_DIR}/r_surf.c
  ${GAME_DIR}/q_shared.c
)

set(REF_SOFT_HEADERS
  ${REF_SOFT_DIR}/adivtab.h
  ${REF_SOFT_DIR}/anorms.h
  ${REF_SOFT_DIR}/asm_draw.h
  ${REF_SOFT_DIR}/d_ifacea.h
  ${REF_SOFT_DIR}/rand1k.h
  ${REF_SOFT_DIR}/r_dither.h
  ${REF_SOFT_DIR}/r_local.h
  ${REF_SOFT_DIR}/r_model.h
)

set(REF_SOFT_SOURCES

)

if(UNIX)
  # Software renderer
  set(REF_SOFT_SOURCES
    ${REF_SOFT_SOURCES}
    ${LINUX_DIR}/r_aclipa.s
    ${LINUX_DIR}/r_draw16.s
    ${LINUX_DIR}/r_drawa.s
    ${LINUX_DIR}/r_edgea.s
    ${LINUX_DIR}/r_scana.s
    ${LINUX_DIR}/r_spr8.s
    ${LINUX_DIR}/r_surf8.s
    ${LINUX_DIR}/math.s
    ${LINUX_DIR}/d_polysa.s
    ${LINUX_DIR}/r_varsa.s
    ${LINUX_DIR}/sys_dosa.s
    ${LINUX_DIR}/q_shlinux.c
    ${LINUX_DIR}/glob.c
  )

  # X11
  set(REF_SOFT_SOURCES
    ${REF_SOFT_SOURCES}
    ${LINUX_DIR}/rw_x11.c
  )
endif()

################################################################################
# REF_GL
################################################################################

# Ref GL
set(REF_GL_SOURCES
  ${REF_GL_DIR}/gl_draw.c
  ${REF_GL_DIR}/gl_image.c
  ${REF_GL_DIR}/gl_light.c
  ${REF_GL_DIR}/gl_mesh.c
  ${REF_GL_DIR}/gl_model.c
  ${REF_GL_DIR}/gl_rmain.c
  ${REF_GL_DIR}/gl_rmisc.c
  ${REF_GL_DIR}/gl_rsurf.c
  ${REF_GL_DIR}/gl_warp.c
  ${GAME_DIR}/q_shared.c
)

set(REF_GL_HEADERS
  ${REF_GL_DIR}/anorms.h
  ${REF_GL_DIR}/anormtab.h
  ${REF_GL_DIR}/gl_local.h
  ${REF_GL_DIR}/gl_model.h
  ${REF_GL_DIR}/qgl.h
  ${REF_GL_DIR}/warpsin.h
)

if(UNIX)
  # Ref GL
  set(REF_GL_SOURCES
    ${REF_GL_SOURCES}
    ${LINUX_DIR}/qgl_linux.c
    ${LINUX_DIR}/gl_glx.c
    ${LINUX_DIR}/q_shlinux.c
    ${LINUX_DIR}/glob.c
  )

  # REF GLX
  set(REF_GL_SOURCES
    ${REF_GL_SOURCES}
    ${LINUX_DIR}/gl_glx.c
  )
endif()

################################################################################
# REF_VK
################################################################################

set(REF_VK_SOURCES
  ${GAME_DIR}/q_shared.c
  ${REF_VK_DIR}/vk_buffer.c
  ${REF_VK_DIR}/vk_cmd.c
  ${REF_VK_DIR}/vk_common.c
  ${REF_VK_DIR}/vk_device.c
  ${REF_VK_DIR}/vk_draw.c
  ${REF_VK_DIR}/vk_image.c
  ${REF_VK_DIR}/vk_light.c
  ${REF_VK_DIR}/vk_mem_alloc.cpp
  ${REF_VK_DIR}/vk_mesh.c
  ${REF_VK_DIR}/vk_model.c
  ${REF_VK_DIR}/vk_pipeline.c
  ${REF_VK_DIR}/vk_rmain.c
  ${REF_VK_DIR}/vk_rmisc.c
  ${REF_VK_DIR}/vk_rsurf.c
  ${REF_VK_DIR}/vk_shaders.c
  ${REF_VK_DIR}/vk_swapchain.c
  ${REF_VK_DIR}/vk_validation.c
  ${REF_VK_DIR}/vk_warp.c
)

set(REF_VK_HEADERS
  ${CLIENT_DIR}/keys.h
  ${COMMON_DIR}/qcommon.h
  ${GAME_DIR}/q_shared.h
  ${REF_VK_DIR}/anorms.h
  ${REF_VK_DIR}/anormtab.h
  ${REF_VK_DIR}/qvk.h
  ${REF_VK_DIR}/vk_local.h
  ${REF_VK_DIR}/vk_mem_alloc.h
  ${REF_VK_DIR}/vk_model.h
  ${REF_VK_DIR}/vk_shaders.h
)

if(UNIX)
  set(REF_VK_SOURCES
    ${REF_VK_SOURCES}
    ${LINUX_DIR}/glob.c
    ${LINUX_DIR}/q_shlinux.c
    ${LINUX_DIR}/vk_imp.c
  )
  set(REF_VK_HEADERS
    ${REF_VK_HEADERS}
    ${CLIENT_DIR}/ref.h
    ${LINUX_DIR}/rw_linux.h
    ${LINUX_DIR}/vk_linux.h
  )
endif()

################################################################################
# SETUP AND BUILD
################################################################################

# Main Quake 2 executable
add_executable(quake2 ${QUAKE2_SOURCES} ${QUAKE2_HEADERS})
set_target_properties(quake2 PROPERTIES
  RUNTIME_OUTPUT_DIRECTORY ${BUILD_DIR}
)
target_link_libraries(quake2 ${LDFLAGS})

# Quake 2 Dedicated Server executable
add_executable(q2ded ${Q2DED_SOURCES} ${Q2DED_HEADERS})
set_target_properties(q2ded PROPERTIES
  RUNTIME_OUTPUT_DIRECTORY ${BUILD_DIR}
)
target_link_libraries(q2ded ${LDFLAGS})
target_compile_definitions(q2ded PRIVATE -DDEDICATED_ONLY)

# Build the game dynamic library
add_library(game MODULE ${GAME_SOURCES} ${GAME_HEADERS})
set_target_properties(game PROPERTIES
    PREFIX ""
    LIBRARY_OUTPUT_DIRECTORY ${BUILD_DIR}/baseq2
    RUNTIME_OUTPUT_DIRECTORY ${BUILD_DIR}/baseq2
    SUFFIX ${ARCH}${SHLIBEXT}
)
target_compile_options(game PRIVATE ${SHLIBLDFLAGS})

# Build the GLX dynamic library
add_library(ref_glx MODULE ${REF_GL_SOURCES} ${REF_GL_HEADERS})
set_target_properties(ref_glx PROPERTIES
    PREFIX ""
    LIBRARY_OUTPUT_DIRECTORY ${BUILD_DIR}
    RUNTIME_OUTPUT_DIRECTORY ${BUILD_DIR}
    SUFFIX ${SHLIBEXT}
)
target_compile_options(ref_glx PRIVATE ${SHLIBLDFLAGS} ${GLXLDFLAGS})

# Build the soft renderer dynamic library
add_library(ref_softx MODULE ${REF_SOFT_SOURCES} ${REF_SOFT_HEADERS})
set_target_properties(ref_softx PROPERTIES
    PREFIX ""
    LIBRARY_OUTPUT_DIRECTORY ${BUILD_DIR}
    RUNTIME_OUTPUT_DIRECTORY ${BUILD_DIR}
    SUFFIX ${SHLIBEXT}
)
target_compile_options(ref_softx PRIVATE ${SHLIBLDFLAGS} ${XLDFLAGS})

# Build the vk renderer dynamic library
add_library(ref_vk MODULE ${REF_VK_SOURCES} ${REF_VK_HEADERS})
set_target_properties(ref_vk PROPERTIES
    PREFIX ""
    LIBRARY_OUTPUT_DIRECTORY ${BUILD_DIR}
    RUNTIME_OUTPUT_DIRECTORY ${BUILD_DIR}
    SUFFIX ${SHLIBEXT}
)
target_include_directories(ref_vk PRIVATE ${VKCFLAGS})
target_compile_options(ref_vk PRIVATE ${SHLIBLDFLAGS})
target_link_libraries(ref_vk ${VKLDFLAGS} ${Vulkan_LIBRARIES})
target_compile_definitions(ref_vk PRIVATE -DVK_LOADER_DEBUG=all)

# Build the ctf renderer dynamic library
add_library(ctf MODULE ${CTF_SOURCES} ${CTF_HEADERS})
set_target_properties(ctf PROPERTIES
    PREFIX ""
    OUTPUT_NAME "game"
    LIBRARY_OUTPUT_DIRECTORY ${BUILD_DIR}/ctf
    RUNTIME_OUTPUT_DIRECTORY ${BUILD_DIR}/ctf
    SUFFIX ${ARCH}${SHLIBEXT}
)
target_compile_options(ctf PRIVATE ${SHLIBLDFLAGS})

# Build the rogue renderer dynamic library
add_library(rogue MODULE ${ROGUE_SOURCES} ${ROGUE_HEADERS})
set_target_properties(rogue PROPERTIES
    PREFIX ""
    OUTPUT_NAME "game"
    LIBRARY_OUTPUT_DIRECTORY ${BUILD_DIR}/rogue
    RUNTIME_OUTPUT_DIRECTORY ${BUILD_DIR}/rogue
    SUFFIX ${ARCH}${SHLIBEXT}
)
target_compile_options(rogue PRIVATE ${SHLIBLDFLAGS})

# Build the xatrix renderer dynamic library
add_library(xatrix MODULE ${XATRIX_SOURCES} ${XATRIX_HEADERS})
set_target_properties(xatrix PROPERTIES
    PREFIX ""
    OUTPUT_NAME "game"
    LIBRARY_OUTPUT_DIRECTORY ${BUILD_DIR}/xatrix
    RUNTIME_OUTPUT_DIRECTORY ${BUILD_DIR}/xatrix
    SUFFIX ${ARCH}${SHLIBEXT}
)
target_compile_options(xatrix PRIVATE ${SHLIBLDFLAGS})

# Build the zaero renderer dynamic library
add_library(zaero MODULE ${ZAERO_SOURCES} ${ZAERO_HEADERS})
set_target_properties(zaero PROPERTIES
    PREFIX ""
    OUTPUT_NAME "game"
    LIBRARY_OUTPUT_DIRECTORY ${BUILD_DIR}/zaero
    RUNTIME_OUTPUT_DIRECTORY ${BUILD_DIR}/zaero
    SUFFIX ${ARCH}${SHLIBEXT}
)
target_compile_options(zaero PRIVATE ${SHLIBLDFLAGS})

