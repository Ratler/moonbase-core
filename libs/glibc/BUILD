  # The fortify compiler option cause the build to fail
  bad_flags -D_FORTIFY_SOURCE=2 -fstack-protector-strong &&

  # 32bit build fails with -Os due to a ebp inline assembly problem
  bad_flags -Os &&
  CFLAGS+=" -O2" &

  # Fix NPTL issues with Xen on i686
  export CFLAGS+=" -mno-tls-direct-seg-refs" &&

  mkdir -p $SOURCE_DIRECTORY/tmp  &&

  export CC_OPTS=( Pipe )  &&

  if [ -n "$MAKES" ] && [ "$MAKES" -gt 1 ] ; then
    sedit "s/# PARALLELMFLAGS = -j 4/PARALLELMFLAGS = -j $MAKES/" Makefile.in
  fi  &&

  export CC=gcc  &&

  # Don't fully strip libc
  if [[ "$LDFLAGS" == *-s* ]] ; then
    STRIP=y
  fi  &&
  unset LDFLAGS  &&

  INSTALL_ROOT=$SOURCE_DIRECTORY/glibcroot  &&
  mkdir -p $INSTALL_ROOT  &&
  cd $INSTALL_ROOT  &&

  ../configure --prefix=/usr              \
               --infodir=/usr/share/info  \
               --mandir=/usr/share/man    \
               --disable-werror           \
               --with-elf                 \
               --without-gd               \
               --without-cvs              \
               --enable-shared            \
               --host=$BUILD              \
               --build=$BUILD             \
               --enable-obsolete-rpc      \
               --disable-multi-arch       \
               --enable-add-ons           \
               --with-__thread            \
               --with-tls                 \
               --disable-profile          \
               --enable-bind-now          \
               --enable-kernel=2.6.32     \
               --with-headers=/usr/include \
               --enable-stackguard-randomization \
               $OPTS  &&

  # build libraries with hardening disabled
  echo "build-programs=no" >> configparms &&
  make &&

  # Re-enable hardening
  sedit '/build-programs=/s;no;yes;' configparms &&
  echo "CC += -fstack-protector-strong -D_FORTIFY_SOURCE=2" >> configparms &&
  echo "CXX += -fstack-protector-strong -D_FORTIFY_SOURCE=2" >> configparms &&
  make &&

  # Preserve the old glibc temporarily
  LD=/lib/ld-linux.so.2         &&
  GLIBC=/lib/libc.so.6          &&
  DL=/lib/libdl.so.2            &&
  M=/lib/libm.so.6              &&
  RT=/lib/librt.so.1            &&
  PTHREAD=/lib/libpthread.so.0  &&
  NSS=/lib/libnss_files.so.2    &&
  UTIL=/lib/libutil.so.1        &&
  NSL=/lib/libnsl.so.1          &&
  CRYPT=/lib/libcrypt.so.1      &&

  case "`arch`" in
    alpha)
          GLIBC=/lib/libc.so.6.1  &&
          DL=/lib/libdl.so.2.1    &&
          M=/lib/libm.so.6.1
          ;;
     ppc)
          LD=/lib/ld.so.1  &&
          DL=/lib/libdl.so.2
          ;;
     *)
          DL=/lib/libdl.so.2
          ;;
  esac  &&

  cp $LD /lib/ld-lunar.so               &&
  cp $GLIBC /lib/libc-lunar.so          &&
  cp $DL /lib/libdl-lunar.so            &&
  cp $M /lib/libm-lunar.so              &&
  cp $RT /lib/librt-lunar.so            &&
  cp $PTHREAD /lib/libpthread-lunar.so  &&
  cp $NSS /lib/libnss_files-lunar.so    &&
  cp $UTIL /lib/libutil-lunar.so        &&
  cp $NSL /lib/libnsl-lunar.so          &&
  cp $CRYPT /lib/libcrypt-lunar.so      &&

  # Setup links for libraries
  sln /lib/ld-lunar.so $LD               &&
  sln /lib/libc-lunar.so $GLIBC          &&
  sln /lib/libdl-lunar.so $DL            &&
  sln /lib/libm-lunar.so $M              &&
  sln /lib/librt-lunar.so $RT            &&
  sln /lib/libpthread-lunar.so $PTHREAD  &&
  sln /lib/libnss_files-lunar.so $NSS    &&
  sln /lib/libutil-lunar.so $UTIL        &&
  sln /lib/libnsl-lunar.so $NSL          &&
  sln /lib/libcrypt-lunar.so $CRYPT      &&

  # clean hack to make sure gnu/stubs.h gets refreshed
  rm -f /usr/include/gnu/stubs.h  &&

  # This removes the old glibc
  prepare_install  &&

  # And now we install the new
  make install  &&

  # Now optionally perform debug symbol stripping only
  if [ "$STRIP" == "y" ] ; then
    for FN in ld-$MAJOR.so \
        lib{anl,BrokenLocale,c,crypt,dl,m,nsl,util,pthread,resolv}-$MAJOR.so \
        libnss_{compat,files,hesiod,nis,nisplus}-$MAJOR.so \
        lib{memusage,pcprofile,SegFault,thread_db-1.0}.so ; do
      strip -S /lib/$FN
    done  &&
    for FN in "/usr/lib/gconv/*.so" ; do
      strip -S $FN
    done
  fi  &&

  # Rebuild the ld cache
  rm -f /etc/ld.so.cache 2>/dev/null  &&
  ldconfig                            &&

  # Remove the temporary libc
  rm /lib/{ld,lib{c,dl,m,rt,pthread,nss_files,util,nsl,crypt}}-lunar.so  &&

  if [ ! -e /etc/nscd.conf ]; then
    install -m644 $SOURCE_DIRECTORY/nscd/nscd.conf /etc/nscd.conf
  fi &&
  if [ ! -e /etc/gai.conf ]; then
    install -m644 $SOURCE_DIRECTORY/posix/gai.conf /etc/gai.conf
  fi &&

  # Generate locale info
  if [ -z "$GLIBC_LOCALES" ]; then
    make localedata/install-locales
  else
    devoke_installwatch  &&
    for LOCALE in $GLIBC_LOCALES; do
      INPUTFILE=$(echo $LOCALE | cut -d/ -f1 | sed -e 's:\..*$::g')  &&
      OUTPUTFILE=$(echo $LOCALE | cut -d/ -f1)  &&
      CHARMAP=$(echo  $LOCALE | cut -d/ -f2)    &&
      mkdir -p /usr/lib/locale/$OUTPUTFILE      &&
      echo "localedef -i $INPUTFILE -f $CHARMAP $OUTPUTFILE"  &&
      localedef -i $INPUTFILE -f $CHARMAP $OUTPUTFILE
    done &&
    invoke_installwatch  &&
    touch /usr/lib/locale/locale-archive
  fi  &&

  # Touch undetected symlinks
  touch -c -h $LD $GLIBC $DL $M $RT $PTHREAD $NSS $UTIL $NSL $CRYPT /lib/lib{BrokenLocale.so.1,anl.so.1,cidn.so.1,nss_compat.so.2,nss_db.so.2,nss_dns.so.2,nss_hesiod.so.2,nss_nis.so.2,nss_nisplus.so.2,resolv.so.2,thread_db.so.1}  &&
  touch -c -h /usr/lib/lib{BrokenLocale,anl,cidn,crypt,dl,m,nsl,nss_compat,nss_db,nss_dns,nss_files,nss_hesiod,nss_nis,nss_nisplus,resolv,rt,thread_db,util}.so  &&

  ldconfig
