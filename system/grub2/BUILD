(
  
  patch_it $SOURCE_CACHE/$SOURCE2 0 &&
  unset CPP CXX CC CFLAGS CXXFLAGS CPPFLAGS LDFLAGS  &&
  
  OPTS+=" --with-platform="pc" \
    --enable-mm-debug \
    --enable-nls \
    --enable-cache-stats \
    --disable-werror \
    --with-bootdir=/boot \
    --with-grubdir=grub " &&
  
  default_build
  
) > $C_FIFO 2>&1
