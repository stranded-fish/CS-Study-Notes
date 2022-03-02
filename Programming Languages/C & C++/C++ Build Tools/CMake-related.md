# CMake

链接第三方库

手工方式：

https://blog.csdn.net/qq_47343729/article/details/109124149

修改 CMakeLists.txt：

如添加 /usr/local/lib/libzstd.a 库

MYSQL_ADD_EXECUTABLE(xtrabackup
  xtrabackup.cc
  innobackupex.cc
  changed_page_bitmap.cc
  compact.cc
  datasink.c
  ds_archive.c
  ds_buffer.c
  ds_compress.c
  ds_encrypt.c
  ds_decrypt.c
  ds_local.c
  ds_stdout.c
  ds_tmpfile.c
  ds_xbstream.c
  fil_cur.cc
  quicklz/quicklz.c
  read_filt.cc
  write_filt.cc
  wsrep.cc
  xbcrypt_common.c
  xbcrypt_write.c
  xbstream_write.c
  backup_mysql.cc
  backup_copy.cc
  keyring.cc
  ../../../../plugin/keyring/keyring_impl.cc
  ../../../../plugin/keyring/keyring_key.cc
  ../../../../plugin/keyring/buffered_file_io.cc
  ../../../../plugin/keyring/keys_container.cc
  ../../../../sql-common/client_authentication.cc
  ../../../../sql/des_key_file.cc
  )

SET_TARGET_PROPERTIES(xtrabackup PROPERTIES ENABLE_EXPORTS TRUE)

TARGET_LINK_LIBRARIES(xtrabackup 
  mysqlserver 
  ${GCRYPT_LIBS} 
  archive_static
  crc
  **/usr/local/lib/libzstd.a**
  )


https://elloop.github.io/tools/2016-04-10/learning-cmake-2-commands
https://www.jianshu.com/p/37fbe3dd202b
