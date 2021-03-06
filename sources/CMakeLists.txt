cmake_minimum_required(VERSION 2.8)
project(all-xtrx C)

# Set the version information here
set(MAJOR_VERSION 0)
set(API_COMPAT    0)
set(MINOR_VERSION 2)
set(MAINT_VERSION git)

set(LIBVER  "${MAJOR_VERSION}.${API_COMPAT}.${MINOR_VERSION}")

add_definitions(-Wall -g)
set(BUILD_SHARED_LIBS ON)

add_subdirectory(libxtrxdsp)
add_subdirectory(libusb3380)

set(LIBUSB3380_FOUND         TRUE CACHE INTERNAL "libusb3380 internal")
set(LIBUSB3380_LIBRARIES     usb3380 CACHE INTERNAL "libusb3380 internal")
set(LIBUSB3380_INCLUDE_DIRS  ${CMAKE_CURRENT_SOURCE_DIR}/libusb3380 CACHE INTERNAL "libusb3380 internal")

add_subdirectory(libxtrxll)
include_directories(libxtrxll)

add_subdirectory(liblms7002m)

#include_directories(libxtrxdsp)

set(LIBLMS7COMPACT_FOUND        TRUE CACHE INTERNAL "liblms7compact internal")
set(LIBLMS7COMPACT_LIBRARIES    lms7compact CACHE INTERNAL "liblms7compact internal")
set(LIBLMS7COMPACT_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/liblms7002m CACHE INTERNAL "liblms7compact internal")


set(LIBXTRXDSP_FOUND        TRUE CACHE INTERNAL "libxtrxdsp internal")
set(LIBXTRXDSP_LIBRARIES    xtrxdsp CACHE INTERNAL "libxtrxdsp internal")
set(LIBXTRXDSP_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/libxtrxdsp CACHE INTERNAL "libxtrxdsp internal")

set(LIBXTRXLL_FOUND         TRUE CACHE INTERNAL "libxtrxll internal")
set(LIBXTRXLL_LIBRARIES     xtrxll CACHE INTERNAL "libxtrxll internal")
set(LIBXTRXLL_INCLUDE_DIRS  ${CMAKE_CURRENT_SOURCE_DIR}/libxtrxll CACHE INTERNAL "libxtrxll internal")


add_subdirectory(libxtrx)
add_subdirectory(xtrx_linux_pcie_drv)

add_subdirectory(docs)

########################################################################
# package generator
########################################################################
if(NOT CPACK_GENERATOR)
    set(CPACK_GENERATOR DEB)
endif()
set(CPACK_PACKAGE_NAME                "libxtrx-all")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "ALL XTRX family libraries")
set(CPACK_PACKAGE_VENDOR              "Fairwaves, Inc.")
set(CPACK_PACKAGE_CONTACT             "http://fairwaves.co/wp/contact-us/")
set(CPACK_PACKAGE_VERSION             ${LIBVER}-1)
set(CPACK_DEBIAN_PACKAGE_DEPENDS      "libc6 (>= 2.17)")
#if(ENABLE_PCIE)
#   set(CPACK_DEBIAN_PACKAGE_DEPENDS      "${CPACK_DEBIAN_PACKAGE_DEPENDS}, dkms (>= 2.1.0)")
#endif(ENABLE_PCIE)
if(ENABLE_SOAPY)
   set(CPACK_DEBIAN_PACKAGE_DEPENDS      "${CPACK_DEBIAN_PACKAGE_DEPENDS}, soapysdr (>= 0.5.2)")
endif(ENABLE_SOAPY)
if(ENABLE_USB3380)
   set(CPACK_DEBIAN_PACKAGE_DEPENDS      "${CPACK_DEBIAN_PACKAGE_DEPENDS}, libusb-1.0-0 (>= 1.0.10)")
endif(ENABLE_USB3380)


set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}_${CPACK_DEBIAN_PACKAGE_ARCHITECTURE}")
set(CPACK_SET_DESTDIR               "")
set(CPACK_PACKAGING_INSTALL_PREFIX  "/usr")
set(CPACK_PACKAGE_CONTACT           "Sergey Kostanbaev <sergey.kostanbaev@fairwaves.co>")

if(ENABLE_USB3380)
  option(INSTALL_UDEV_RULES "Install udev rules for xtrx-usb" OFF)
  if (INSTALL_UDEV_RULES)
      set(UDEV_RULES_PATH
          "/etc/udev/rules.d"
          CACHE STRING
          "Target directory for udev rule installation. Ensure you have permissions to write to this directory."
          )
      install (
          FILES libxtrxll/mod_usb3380/udev/50-xtrx-usb3380.rules
          DESTINATION ${UDEV_RULES_PATH}
          COMPONENT "xtrx-usb"
          )
  else (INSTALL_UDEV_RULES)
      message (STATUS "Udev rules not being installed, install them with -DINSTALL_UDEV_RULES=ON")
  endif (INSTALL_UDEV_RULES)
endif(ENABLE_USB3380)

#########################################################################
# DKMS intallation
#########################################################################
#if(ENABLE_PCIE)
#  set(CPACK_DEBIAN_PACKAGE_CONTROL_EXTRA "${CMAKE_CURRENT_SOURCE_DIR}/debian/postinst;${CMAKE_CURRENT_SOURCE_DIR}/debian/prerm;" )
#
#  set(DKMSVER "0.0.1-1")
#  install(FILES helpers/50-xtrx.rules DESTINATION "../etc/udev/rules.d" COMPONENT xtrx-drv)
#  install(FILES linux/Makefile linux/dkms.conf linux/xtrx.c linux/xtrx_defs.h  DESTINATION "src/xtrx-${DKMSVER}" COMPONENT xtrx-drv)
#  install(FILES xtrxll_regs.vh DESTINATION "src/xtrx-${DKMSVER}" COMPONENT xtrx-drv)
#endif(ENABLE_PCIE)

include(CPack) #include last
