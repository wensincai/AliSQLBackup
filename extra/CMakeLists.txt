# Copyright (c) 2006, 2012, Oracle and/or its affiliates. All rights reserved.
# 
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
# 
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
# 
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

INCLUDE_DIRECTORIES(
${CMAKE_SOURCE_DIR}/include 
${ZLIB_INCLUDE_DIR}
# Following is for perror, in case NDB is compiled in.
${CMAKE_SOURCE_DIR}/storage/ndb/include
${CMAKE_SOURCE_DIR}/storage/ndb/include/util  
${CMAKE_SOURCE_DIR}/storage/ndb/include/ndbapi
${CMAKE_SOURCE_DIR}/storage/ndb/include/portlib
${CMAKE_SOURCE_DIR}/storage/ndb/include/mgmapi)

# Default install component for the files is Server here
SET(MYSQL_INSTALL_COMPONENT Server)

IF(NOT CMAKE_CROSSCOMPILING)
 ADD_EXECUTABLE(comp_err comp_err.c)
 TARGET_LINK_LIBRARIES(comp_err mysys mysys_ssl)
 SET_TARGET_PROPERTIES(comp_err PROPERTIES LINKER_LANGUAGE CXX)
ENDIF()


ADD_CUSTOM_COMMAND(OUTPUT ${PROJECT_BINARY_DIR}/include/mysqld_error.h 
                   ${PROJECT_BINARY_DIR}/sql/share/english/errmsg.sys
                   COMMAND comp_err
                    --charset=${PROJECT_SOURCE_DIR}/sql/share/charsets 
                    --out-dir=${PROJECT_BINARY_DIR}/sql/share/
                    --header_file=${PROJECT_BINARY_DIR}/include/mysqld_error.h 
                    --name_file=${PROJECT_BINARY_DIR}/include/mysqld_ername.h
                    --state_file=${PROJECT_BINARY_DIR}/include/sql_state.h
                    --in_file=${PROJECT_SOURCE_DIR}/sql/share/errmsg-utf8.txt
                   DEPENDS ${PROJECT_SOURCE_DIR}/sql/share/errmsg-utf8.txt
                   ${CMAKE_CURRENT_SOURCE_DIR}/comp_err.c)

ADD_CUSTOM_TARGET(GenError
                  ALL
                  DEPENDS 
                  ${PROJECT_BINARY_DIR}/include/mysqld_error.h
                  ${PROJECT_BINARY_DIR}/sql/share/english/errmsg.sys 
                  ${PROJECT_SOURCE_DIR}/sql/share/errmsg-utf8.txt)

## Most executables depend on libeay32.dll (through mysys_ssl).
## For 'comp_err' we need it to be available during build.
COPY_OPENSSL_DLLS(copy_openssl_extra)
IF (WIN32 AND WITH_SSL_PATH AND HAVE_CRYPTO_DLL)
  ADD_DEPENDENCIES(GenError copy_openssl_extra)
ENDIF()
