
  patch_it $SOURCE2 1 &&
  patch_it $SOURCE3 1 &&
  patch_it $SOURCE4 1 &&

# This lives in /usr/bin/
  sedit "s:/bin/ls:/usr/bin/ls:g" src/corelib/global/global.pri src/3rdparty/webkit/Source/WebCore/loader/FTPDirectoryParser.h src/3rdparty/webkit/Source/WebCore/loader/FTPDirectoryParser.cpp &&

# Fixes a parsing error. See https://bugreports.qt-project.org/browse/QTBUG-22829 and 
# https://bugs.kde.org/show_bug.cgi?id=304111 Comment 1.
  sed -i "190i pp.macros[\"BOOST_TT_HAS_OPERATOR_HPP_INCLUDED\"]; // rh#756395" src/tools/moc/main.cpp &&

# By ommitting the --prefix switch from the configure and performing this sedit we can remove
# the group of sedit to repoint moc, uic and rcc locations to /usr/bin. 
  sedit "s:/usr/local/Trolltech/:$MODULE_PREFIX/share/$MODULE/Trolltech/:g" configure &&

# Needed so it can find some of the source libs during make.
  export LD_LIBRARY_PATH=$SOURCE_DIRECTORY/lib:${LD_LIBRARY_PATH} &&

  if [[ -d /opt/lunar/qt ]] ; then
    ld_remove /opt/lunar/qt/lib
  fi &&

  if [[ -d /usr/lib/$MODULE ]] ; then
    ld_remove /usr/lib/$MODULE
  fi &&

  if [[ $LICENSE_TYPE == "y" ]] ; then
    LICENSE_TYPE="-opensource"
     else
    LICENSE_TYPE="-commercial"
  fi &&

  sedit "s/-O2/$CFLAGS/" mkspecs/common/gcc-base.conf &&
  sedit "s:-Wl,-rpath,::" mkspecs/common/gcc-base-unix.conf &&
  sed -i "/^QMAKE_LFLAGS\s/s|+=|+= ${LDFLAGS}|g" mkspecs/common/gcc-base.conf &&

# Temporary. Make failes if no jit. Will revisit on next bump. Found by keios.
  OPTS+=" -scripttools -no-declarative-debug -release" &&

  ./configure  -confirm-license "${LICENSE_TYPE}"                  \
               -bindir "${MODULE_PREFIX}/bin"                      \
               -libdir "${MODULE_PREFIX}/lib/$MODULE"              \
               -docdir "${MODULE_PREFIX}/share/$MODULE"            \
               -headerdir "${MODULE_PREFIX}/include/$MODULE"       \
               -plugindir "${MODULE_PREFIX}/lib/$MODULE/plugins"   \
               -importdir "${MODULE_PREFIX}/share/doc/$MODULE"     \
               -datadir "${MODULE_PREFIX}/share/$MODULE"           \
               -translationdir "${MODULE_PREFIX}/share/$MODULE"    \
               -sysconfdir "/etc/xdg"                              \
               -examplesdir "${MODULE_PREFIX}/share/doc/$MODULE"   \
               -demosdir "${MODULE_PREFIX}/share/doc/$MODULE"      \
               $OPTS                                              &&

  default_make &&

# Still need the bin dir here for those modules like google-gadgets-for-linux that
# want to use this path.
  mkdir -p /usr/share/qt4/Trolltech/Qt-`lvu version qt4`/bin &&
  ln -sf /usr/bin/moc /usr/share/qt4/Trolltech/Qt-`lvu version qt4`/bin/moc &&
  ln -sf /usr/bin/qmake /usr/share/qt4/Trolltech/Qt-`lvu version qt4`/bin/qmake &&
  ln -sf /usr/bin/uic /usr/share/qt4/Trolltech/Qt-`lvu version qt4`/bin/uic &&
 
# It wants to point moc_location and uic_location to ${prefix}/src/qt-everywhere-opensource-src-4.8.4/bin
# and after much fiddling cannot get it to point them to /usr/bin. So lets just sedit these rascals.
  for i in `ls /usr/lib/$MODULE/pkgconfig/Q*.pc` ; do sed -i "s:/usr/src/qt-everywhere-opensource-src-$VERSION:/usr:" $i ; done &&
  if [ -f /usr/lib/$MODULE/pkgconfig/phonon.pc ]; then
  sed -i "s:/usr/src/qt-everywhere-opensource-src-$VERSION/:/usr:" /usr/lib/$MODULE/pkgconfig/phonon.pc
  fi &&
# Like for moc and uic, the same needs doing for the *.prl files.
  find /usr/lib/qt4 -type f -name '*.prl' \
      -exec sed -i -e '/^QMAKE_PRL_BUILD_DIR/d;s/\(QMAKE_PRL_LIBS =\).*/\1/' {} \; &&

  if [[ "$MAKE_EXAMPLE_DEMOS" == "y" ]] ; then
  cat > ${MODULE}-designer.desktop << EOF  &&
[Desktop Entry]
Type=Application
Name=Qt4 Designer
Comment=Design GUIs for Qt4 applications
Exec=${MODULE_PREFIX}/bin/designer
Icon=${MODULE_PREFIX}/share/doc/qt4/widgets/icons/images/designer.png
Categories=Qt;Development;
Terminal=false
EOF
  install -D -m644 ${MODULE}-designer.desktop ${MODULE_PREFIX}/share/applications/${MODULE}-designer.desktop 

  cat > ${MODULE}-assistant.desktop << EOF  &&
[Desktop Entry]
Type=Application
Name=Qt4 Assistant
Comment=Shows Qt4 documentation and examples
Exec=${MODULE_PREFIX}/bin/assistant
Icon=${MODULE_PREFIX}/share/qt4/src/images/assistant-assistant.png
Categories=Qt;Development;Documentation;
Terminal=false
EOF
  install -D -m644 ${MODULE}-assistant.desktop ${MODULE_PREFIX}/share/applications/${MODULE}-assistant.desktop

  cat > ${MODULE}-qtdemo.desktop << EOF  &&
[Desktop Entry]
Name=Qt4 Demo 
Comment=Show Qt4 demos and programming examples
Exec=${MODULE_PREFIX}/bin/qtdemo
Icon=${MODULE_PREFIX}/share/$MODULE/src/images/qt-logo.png
Terminal=false
Type=Application
Categories=Qt;Development;
EOF
  install -D -m644 ${MODULE}-qtdemo.desktop ${MODULE_PREFIX}/share/applications/${MODULE}-qtdemo.desktop
fi &&

  cat > ${MODULE}-qtconfig.desktop << EOF  &&
[Desktop Entry]
Name=Qt4 Config 
Comment=Configure Qt4 behavior, styles, fonts
Exec=${MODULE_PREFIX}/bin/qtconfig
Icon=${MODULE_PREFIX}/share/$MODULE/src/images/qt-logo.png
Terminal=false
Type=Application
Categories=Qt;Settings;
EOF
  install -D -m644 ${MODULE}-qtconfig.desktop ${MODULE_PREFIX}/share/applications/${MODULE}-qtconfig.desktop &&

  cat > ${MODULE}-linguist.desktop << EOF  &&
[Desktop Entry]
Name=Qt4 Linquist
Comment=Add translations to Qt4 applications
Exec=${MODULE_PREFIX}/bin/linguist
Icon=${MODULE_PREFIX}/share/$MODULE/src/images/linguist-linguist.png
Terminal=false
Type=Application
Categories=Qt;Settings;
EOF
  install -D -m644 ${MODULE}-linguist.desktop ${MODULE_PREFIX}/share/applications/${MODULE}-linguist.desktop &&

  echo "export QTDIR=\"${MODULE_PREFIX}\"" > $SOURCE_DIRECTORY/$MODULE.rc &&
  echo export QT_PLUGIN_PATH=\"'${QTDIR}'/lib/${MODULE}/plugins\" >> $SOURCE_DIRECTORY/$MODULE.rc &&
  echo export XDG_DATA_DIRS=\"'${XDG_DATA_DIRS:-/usr/share}':/usr/share/$MODULE\" >> $SOURCE_DIRECTORY/$MODULE.rc &&
  echo export PKG_CONFIG_PATH=\"'${QTDIR}'/lib/${MODULE}/pkgconfig:'${PKG_CONFIG_PATH}'\" >> $SOURCE_DIRECTORY/$MODULE.rc &&

  install -m644 $SOURCE_DIRECTORY/$MODULE.rc /etc/profile.d/
