(

  patch_it $SOURCE2 1 &&
  patch_it $SOURCE3 1 &&

  . /etc/profile.d/qt4.rc &&
  . /etc/profile.d/kde4.rc &&

  OPTS+=" -DK3B_ENABLE_HAL_SUPPORT=OFF" &&

  default_cmake_build

) > $C_FIFO 2>&1
