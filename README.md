# $FreeBSD$

freebsd_instance:
  image: freebsd-12-1-release-amd64

env:
  CIRRUS_CLONE_DEPTH: 1

task:
  install_script:
  - pkg install -y
    v4l_compat swig30 ffmpeg curl dbus fdk-aac fontconfig
    freetype2 jackit jansson luajit mbedtls pulseaudio speexdsp
    libsysinfo libudev-devd libv4l libx264 cmake ninja
    mesa-libs lua52 pkgconf
    qt5-svg qt5-qmake qt5-buildtools qt5-x11extras qt5-xml
  script:
  - mkdir build
  - cd build
  - cmake -DUNIX_STRUCTURE=1 -GNinja ..
  - ninja
# please use clang-format version 8 or later

Standard: Cpp11
AccessModifierOffset: -8
AlignAfterOpenBracket: Align
AlignConsecutiveAssignments: false
AlignConsecutiveDeclarations: false
AlignEscapedNewlines: Left
AlignOperands: true
AlignTrailingComments: true
#AllowAllArgumentsOnNextLine: false  # requires clang-format 9
#AllowAllConstructorInitializersOnNextLine: false  # requires clang-format 9
AllowAllParametersOfDeclarationOnNextLine: false
AllowShortBlocksOnASingleLine: false
AllowShortCaseLabelsOnASingleLine: false
AllowShortFunctionsOnASingleLine: Inline
AllowShortIfStatementsOnASingleLine: false
#AllowShortLambdasOnASingleLine: Inline  # requires clang-format 9
AllowShortLoopsOnASingleLine: false
AlwaysBreakAfterDefinitionReturnType: None
AlwaysBreakAfterReturnType: None
AlwaysBreakBeforeMultilineStrings: false
AlwaysBreakTemplateDeclarations: false
BinPackArguments: true
BinPackParameters: true
BraceWrapping:
  AfterClass: false
  AfterControlStatement: false
  AfterEnum: false
  AfterFunction: true
  AfterNamespace: false
  AfterObjCDeclaration: false
  AfterStruct: false
  AfterUnion: false
  AfterExternBlock: false
  BeforeCatch: false
  BeforeElse: false
  IndentBraces: false
  SplitEmptyFunction: true
  SplitEmptyRecord: true
  SplitEmptyNamespace: true
BreakBeforeBinaryOperators: None
BreakBeforeBraces: Custom
BreakBeforeTernaryOperators: true
BreakConstructorInitializers: BeforeColon
BreakStringLiterals: false  # apparently unpredictable
ColumnLimit: 80
CompactNamespaces: false
ConstructorInitializerAllOnOneLineOrOnePerLine: true
ConstructorInitializerIndentWidth: 8
ContinuationIndentWidth: 8
Cpp11BracedListStyle: true
DerivePointerAlignment: false
DisableFormat: false
FixNamespaceComments: false
ForEachMacros: 
  - 'json_object_foreach'
  - 'json_object_foreach_safe'
  - 'json_array_foreach'
IncludeBlocks: Preserve
IndentCaseLabels: false
IndentPPDirectives: None
IndentWidth: 8
IndentWrappedFunctionNames: false
KeepEmptyLinesAtTheStartOfBlocks: true
MaxEmptyLinesToKeep: 1
NamespaceIndentation: None
#ObjCBinPackProtocolList: Auto  # requires clang-format 7
ObjCBlockIndentWidth: 8
ObjCSpaceAfterProperty: true
ObjCSpaceBeforeProtocolList: true

PenaltyBreakAssignment: 10
PenaltyBreakBeforeFirstCallParameter: 30
PenaltyBreakComment: 10
PenaltyBreakFirstLessLess: 0
PenaltyBreakString: 10
PenaltyExcessCharacter: 100
PenaltyReturnTypeOnItsOwnLine: 60

PointerAlignment: Right
ReflowComments: false
SortIncludes: false
SortUsingDeclarations: false
SpaceAfterCStyleCast: false
#SpaceAfterLogicalNot: false  # requires clang-format 9
SpaceAfterTemplateKeyword: false
SpaceBeforeAssignmentOperators: true
#SpaceBeforeCtorInitializerColon: true  # requires clang-format 7
#SpaceBeforeInheritanceColon: true  # requires clang-format 7
SpaceBeforeParens: ControlStatements
#SpaceBeforeRangeBasedForLoopColon: true  # requires clang-format 7
SpaceInEmptyParentheses: false
SpacesBeforeTrailingComments: 1
SpacesInAngles: false
SpacesInCStyleCastParentheses: false
SpacesInContainerLiterals: false
SpacesInParentheses: false
SpacesInSquareBrackets: false
#StatementMacros:  # requires clang-format 8
#  - 'Q_OBJECT'
TabWidth: 8
#TypenameMacros:  # requires clang-format 9
#  - 'DARRAY'
UseTab: ForContinuationAndIndentation
---
Language: ObjC
# EditorConfig is awesome: http://EditorConfig.org
# Since OBS follows the Linux kernel coding style I have started this file to
# help with automatically setting various text editors to those requirements.

# top-most EditorConfig file
root = true

# Unix-style newlines with a newline ending every file.
[*]
insert_final_newline = true
trim_trailing_whitespace = true
charset = utf-8
indent_style = tab
indent_size = 8

# As per notr1ch, for 3rd party code that's a part of obs-outputs.
[plugins/obs-outputs/librtmp/*.{cpp,c,h}]
indent_style = space
indent_size = 4
f53df7da64d2dfc542c24656720b2f47c8957164
* text=auto

*.sln             text eol=crlf
*.vcproj          text eol=crlf
*.vcxproj         text eol=crlf
*.vcxproj         text eol=crlf
*.vcxproj.filters text eol=crlf

cmake/ALL_BUILD.vcxproj.user.in text eol=crlf

open_collective: obsproject
patreon: obsproject
name: Clang Format Check

on: [push, pull_request]
jobs:
  ubuntu64:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          submodules: 'recursive'

      - name: Install clang format
        run: |
          # gets us newer clang
          sudo bash -c "cat >> /etc/apt/sources.list" << LLVMAPT
          # 3.8
          deb http://apt.llvm.org/xenial/ llvm-toolchain-xenial-8 main
          deb-src http://apt.llvm.org/xenial/ llvm-toolchain-xenial-8 main
          LLVMAPT

          wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -    

          sudo apt-get -qq update

          sudo apt-get install -y clang-format-8

      - name: Check the Formatting
        run: |
          ./formatcode.sh
          ./CI/check-format.sh

  macos64:
    runs-on: macos-latest        
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          submodules: 'recursive'

      - name: Install clang-format
        run: |
          brew install clang-format

      - name: Check the Formatting
        run: |
          ./formatcode.sh
          ./CI/check-format.sh 

name: 'CI Multiplatform Build'

on:
  push:
    branches:
      - master
  pull_request:
    paths-ignore:
      - '**.md'
    branches:
      - master

env:
  CEF_BUILD_VERSION: '3770'
  CEF_VERSION: '75.1.16+g16a67c4+chromium-75.0.3770.100'

jobs:
  macos64:
    name: 'macOS 64-bit'
    runs-on: [macos-latest]
    env:
      MACOS_DEPS_VERSION: '2020-04-24'
      VLC_VERSION: '3.0.8'
      SPARKLE_VERSION: '1.23.0'
      QT_VERSION: '5.14.1'
    steps:
      - name: 'Checkout'
        uses: actions/checkout@v2
        with:
          submodules: 'recursive'
      - name: 'Fetch Git Tags'
        run: |
          git fetch --prune --unshallow
          echo ::set-env name=OBS_GIT_BRANCH::$(git rev-parse --abbrev-ref HEAD)
          echo ::set-env name=OBS_GIT_HASH::$(git rev-parse --short HEAD)
          echo ::set-env name=OBS_GIT_TAG::$(git describe --tags --abbrev=0)
      - name: 'Install prerequisites (Homebrew)'
        shell: bash
        run: |
          if [ -d "$(brew --cellar)/swig" ]; then
               brew unlink swig
          fi

          if [ -d "$(brew --cellar)/qt" ]; then
               brew unlink qt
          fi

          brew bundle --file ./CI/scripts/macos/Brewfile
      - name: 'Restore Chromium Embedded Framework from cache'
        id: cef-cache
        uses: actions/cache@v1
        env:
          CACHE_NAME: 'cef-cache'
        with:
          path: ${{ github.workspace }}/cmbuild/cef_binary_${{ env.CEF_BUILD_VERSION }}_macosx64
          key: ${{ runner.os }}-pr-${{ env.CACHE_NAME }}-${{ env.CEF_BUILD_VERSION }}
      - name: 'Restore pre-built dependencies from cache'
        id: deps-cache
        uses: actions/cache@v1
        env:
          CACHE_NAME: 'deps-cache'
        with:
          path: /tmp/obsdeps
          key: ${{ runner.os }}-pr-${{ env.CACHE_NAME }}-${{ env.MACOS_DEPS_VERSION }}
      - name: 'Restore VLC dependency from cache'
        id: vlc-cache
        uses: actions/cache@v1
        env:
          CACHE_NAME: 'vlc-cache'
        with:
          path: ${{ github.workspace }}/cmbuild/vlc-${{ env.VLC_VERSION }}
          key: ${{ runner.os }}-pr-${{ env.CACHE_NAME }}-${{ env.VLC_VERSION }}
      - name: 'Restore Sparkle dependency from cache'
        id: sparkle-cache
        uses: actions/cache@v1
        env:
          CACHE_NAME: 'sparkle-cache'
        with:
          path: ${{ github.workspace }}/cmbuild/sparkle
          key: ${{ runner.os }}-pr-${{ env.CACHE_NAME }}-${{ env.SPARKLE_VERSION }}
      - name: 'Install prerequisite: Pre-built dependencies'
        if: steps.deps-cache.outputs.cache-hit != 'true'
        shell: bash
        run: |
          curl -L -O https://github.com/obsproject/obs-deps/releases/download/${{ env.MACOS_DEPS_VERSION }}/osx-deps-${{ env.MACOS_DEPS_VERSION }}.tar.gz
          tar -xf ./osx-deps-${{ env.MACOS_DEPS_VERSION }}.tar.gz -C "/tmp"
      - name: 'Install prerequisite: VLC'
        if: steps.vlc-cache.outputs.cache-hit != 'true'
        shell: bash
        run: |
          curl -L -O https://downloads.videolan.org/vlc/${{ env.VLC_VERSION }}/vlc-${{ env.VLC_VERSION }}.tar.xz
          if [ ! -d "${{ github.workspace }}/cmbuild" ]; then mkdir "${{ github.workspace }}/cmbuild"; fi
          tar -xf ./vlc-${{ env.VLC_VERSION }}.tar.xz -C "${{ github.workspace }}/cmbuild"
      - name: 'Install prerequisite: Sparkle'
        if: steps.sparkle-cache.outputs.cache-hit != 'true'
        shell: bash
        run: |
          curl -L -o sparkle.tar.bz2 https://github.com/sparkle-project/Sparkle/releases/download/${{ env.SPARKLE_VERSION }}/Sparkle-${{ env.SPARKLE_VERSION }}.tar.bz2
          mkdir ${{ github.workspace }}/cmbuild/sparkle
          tar -xf ./sparkle.tar.bz2 -C ${{ github.workspace }}/cmbuild/sparkle
      - name: 'Setup prerequisite: Sparkle'
        shell: bash
        run: sudo cp -R ${{ github.workspace }}/cmbuild/sparkle/Sparkle.framework /Library/Frameworks/Sparkle.framework
      - name: 'Install prerequisite: Chromium Embedded Framework'
        if: steps.cef-cache.outputs.cache-hit != 'true'
        shell: bash
        run: |
          curl -L -O https://obs-nightly.s3-us-west-2.amazonaws.com/cef_binary_${{ env.CEF_BUILD_VERSION }}_macosx64.tar.bz2
          tar -xf ./cef_binary_${{ env.CEF_BUILD_VERSION }}_macosx64.tar.bz2 -C ${{ github.workspace }}/cmbuild/
          cd ${{ github.workspace }}/cmbuild/cef_binary_${{ env.CEF_BUILD_VERSION }}_macosx64
          sed -i '.orig' '/add_subdirectory(tests\/ceftests)/d' ./CMakeLists.txt
          # target 10.11
          sed -i '.orig' s/\"10.9\"/\"10.11\"/ ./cmake/cef_variables.cmake
          mkdir build && cd build
          cmake  -DCMAKE_CXX_FLAGS="-std=c++11 -stdlib=libc++" -DCMAKE_EXE_LINKER_FLAGS="-std=c++11 -stdlib=libc++" -DCMAKE_OSX_DEPLOYMENT_TARGET=10.11  ..
          make -j4
          mkdir libcef_dll
          cd ../../
      - name: 'Configure'
        shell: bash
        run: |
          mkdir ./build
          cd ./build
          cmake -DENABLE_SPARKLE_UPDATER=ON -DCMAKE_OSX_DEPLOYMENT_TARGET=10.11 -DQTDIR="/usr/local/Cellar/qt/${{ env.QT_VERSION }}" -DDepsPath="/tmp/obsdeps" -DVLCPath="${{ github.workspace }}/cmbuild/vlc-${{ env.VLC_VERSION }}" -DBUILD_BROWSER=ON -DBROWSER_DEPLOY=ON -DBUILD_CAPTIONS=ON -DWITH_RTMPS=ON -DCEF_ROOT_DIR="${{ github.workspace }}/cmbuild/cef_binary_${{ env.CEF_BUILD_VERSION }}_macosx64" ..
      - name: 'Build'
        shell: bash
        working-directory: ${{ github.workspace }}/build
        run: make -j4
      - name: 'Install prerequisite: Packages app'
        if: success() && (github.event_name != 'pull_request' || contains( github.event.pull_request.labels.*.name, 'Seeking Testers'))
        shell: bash
        run: |
          curl -L -O https://s3-us-west-2.amazonaws.com/obs-nightly/Packages.pkg
          sudo installer -pkg ./Packages.pkg -target /
      - name: 'Install prerequisite: DMGbuild'
        if: success() && (github.event_name != 'pull_request' || contains( github.event.pull_request.labels.*.name, 'Seeking Testers'))
        shell: bash
        run: |
          pip3 install dmgbuild
      - name: 'Create macOS application bundle'
        if: success() && (github.event_name != 'pull_request' || contains( github.event.pull_request.labels.*.name, 'Seeking Testers'))
        working-directory: ${{ github.workspace }}/build
        shell: bash
        run: |
          if [ -d ./OBS.app ]; then rm -rf ./OBS.app; fi
          mkdir -p OBS.app/Contents/MacOS
          mkdir OBS.app/Contents/PlugIns
          mkdir OBS.app/Contents/Resources

          cp -R rundir/RelWithDebInfo/bin/ ./OBS.app/Contents/MacOS
          cp -R rundir/RelWithDebInfo/data ./OBS.app/Contents/Resources
          cp ../CI/scripts/macos/app/obs.icns ./OBS.app/Contents/Resources
          cp -R rundir/RelWithDebInfo/obs-plugins/ ./OBS.app/Contents/PlugIns
          cp ../CI/scripts/macos/app/Info.plist ./OBS.app/Contents

          if [ -d ./OBS.app/Contents/Resources/data/obs-scripting ]; then
            mv ./OBS.app/Contents/Resources/data/obs-scripting/obslua.so ./OBS.app/Contents/MacOS/
            mv ./OBS.app/Contents/Resources/data/obs-scripting/_obspython.so ./OBS.app/Contents/MacOS/
            mv ./OBS.app/Contents/Resources/data/obs-scripting/obspython.py ./OBS.app/Contents/MacOS/
            rm -rf ./OBS.app/Contents/Resources/data/obs-scripting/
          fi

          install_name_tool -change libmbedtls.12.dylib @executable_path/../Frameworks/libmbedtls.12.dylib ./OBS.app/Contents/Plugins/obs-outputs.so
          install_name_tool -change libmbedcrypto.3.dylib @executable_path/../Frameworks/libmbedcrypto.3.dylib ./OBS.app/Contents/Plugins/obs-outputs.so
          install_name_tool -change libmbedx509.0.dylib @executable_path/../Frameworks/libmbedx509.0.dylib ./OBS.app/Contents/Plugins/obs-outputs.so

          ../CI/scripts/macos/app/dylibBundler -cd -of -a ./OBS.app -q -f \
            -s ./OBS.app/Contents/MacOS \
            -s "${{ github.workspace }}/cmbuild/sparkle/Sparkle.framework" \
            -x ./OBS.app/Contents/PlugIns/coreaudio-encoder.so \
            -x ./OBS.app/Contents/PlugIns/decklink-ouput-ui.so \
            -x ./OBS.app/Contents/PlugIns/frontend-tools.so \
            -x ./OBS.app/Contents/PlugIns/image-source.so \
            -x ./OBS.app/Contents/PlugIns/linux-jack.so \
            -x ./OBS.app/Contents/PlugIns/mac-avcapture.so \
            -x ./OBS.app/Contents/PlugIns/mac-capture.so \
            -x ./OBS.app/Contents/PlugIns/mac-decklink.so \
            -x ./OBS.app/Contents/PlugIns/mac-syphon.so \
            -x ./OBS.app/Contents/PlugIns/mac-vth264.so \
            -x ./OBS.app/Contents/PlugIns/obs-browser.so \
            -x ./OBS.app/Contents/PlugIns/obs-browser-page \
            -x ./OBS.app/Contents/PlugIns/obs-ffmpeg.so \
            -x ./OBS.app/Contents/PlugIns/obs-filters.so \
            -x ./OBS.app/Contents/PlugIns/obs-transitions.so \
            -x ./OBS.app/Contents/PlugIns/obs-vst.so \
            -x ./OBS.app/Contents/PlugIns/rtmp-services.so \
            -x ./OBS.app/Contents/MacOS/obs-ffmpeg-mux \
            -x ./OBS.app/Contents/MacOS/obslua.so \
            -x ./OBS.app/Contents/MacOS/_obspython.so \
            -x ./OBS.app/Contents/PlugIns/obs-x264.so \
            -x ./OBS.app/Contents/PlugIns/text-freetype2.so \
            -x ./OBS.app/Contents/PlugIns/obs-libfdk.so \
            -x ./OBS.app/Contents/PlugIns/obs-outputs.so

          mv ./OBS.app/Contents/MacOS/libobs-opengl.so ./OBS.app/Contents/Frameworks

          sudo cp -R "${{ github.workspace }}/cmbuild/cef_binary_${{ env.CEF_BUILD_VERSION }}_macosx64/Release/Chromium Embedded Framework.framework" ./OBS.app/Contents/Frameworks/
          sudo chown -R $(whoami) ./OBS.app/Contents/Frameworks/
          install_name_tool -change /usr/local/Cellar/qt/${{ env.QT_VERSION }}/QtGui.framework/Versions/5/QtGui @executable_path/../Frameworks/QtGui.framework/Versions/5/QtGui ./OBS.app/Contents/Plugins/obs-browser.so
          install_name_tool -change /usr/local/Cellar/qt/${{ env.QT_VERSION }}/lib/QtCore.framework/Versions/5/QtCore @executable_path/../Frameworks/QtCore.framework/Versions/5/QtCore ./OBS.app/Contents/Plugins/obs-browser.so
          install_name_tool -change /usr/local/Cellar/qt/${{ env.QT_VERSION }}/lib/QtWidgets.framework/Versions/5/QtWidgets @executable_path/../Frameworks/QtWidgets.framework/Versions/5/QtWidgets ./OBS.app/Contents/Plugins/obs-browser.so

          cp ../CI/scripts/macos/app/OBSPublicDSAKey.pem ./OBS.app/Contents/Resources

          plutil -insert CFBundleVersion -string ${{ env.OBS_GIT_TAG }}-${{ env.OBS_GIT_HASH }} ./OBS.app/Contents/Info.plist
          plutil -insert CFBundleShortVersionString -string ${{ env.OBS_GIT_TAG }}-${{ env.OBS_GIT_HASH }} ./OBS.app/Contents/Info.plist
          plutil -insert OBSFeedsURL -string https://obsproject.com/osx_update/feeds.xml ./OBS.app/Contents/Info.plist
          plutil -insert SUFeedURL -string https://obsproject.com/osx_update/stable/updates.xml ./OBS.app/Contents/Info.plist
          plutil -insert SUPublicDSAKeyFile -string OBSPublicDSAKey.pem ./OBS.app/Contents/Info.plist
      - name: 'Package'
        if: success() && (github.event_name != 'pull_request' || contains( github.event.pull_request.labels.*.name, 'Seeking Testers'))
        working-directory: ${{ github.workspace }}/build
        shell: bash
        run: |
          FILE_DATE=$(date +%Y-%m-%d)
          FILE_NAME=$FILE_DATE-${{ env.OBS_GIT_HASH }}-${{ env.OBS_GIT_TAG }}-macOS.dmg
          echo "::set-env name=FILE_NAME::${FILE_NAME}"

          cp ../CI/scripts/macos/package/settings.json.template ./settings.json
          sed -i '' 's#\$\$VERSION\$\$#${{ env.OBS_GIT_TAG }}#g' ./settings.json
          sed -i '' 's#\$\$CI_PATH\$\$#../CI/scripts/macos#g' ./settings.json
          sed -i '' 's#\$\$BUNDLE_PATH\$\$#${{ github.workspace }}/build#g' ./settings.json

          dmgbuild "OBS-Studio ${{ env.OBS_GIT_TAG }}" "${FILE_NAME}" -s ./settings.json
          mkdir ../nightly
          sudo mv ./${FILE_NAME} ../nightly/${FILE_NAME}

      - name: 'Publish'
        if: success() && (github.event_name != 'pull_request' || contains( github.event.pull_request.labels.*.name, 'Seeking Testers'))
        uses: actions/upload-artifact@v2-preview
        with:
          name: '${{ env.FILE_NAME }}'
          path: ./nightly/*.dmg
  ubuntu64:
    name: 'Linux/Ubuntu 64-bit'
    runs-on: [ubuntu-latest]
    steps:
      - name: 'Checkout'
        uses: actions/checkout@v2
        with:
          submodules: 'recursive'
      - name: 'Fetch Git Tags'
        run: |
          git fetch --prune --unshallow
          echo ::set-env name=OBS_GIT_BRANCH::$(git rev-parse --abbrev-ref HEAD)
          echo ::set-env name=OBS_GIT_HASH::$(git rev-parse --short HEAD)
          echo ::set-env name=OBS_GIT_TAG::$(git describe --tags --abbrev=0)
      - name: Install prerequisites (Apt)
        shell: bash
        run: |
          sudo dpkg --add-architecture amd64
          sudo apt-get -qq update
          sudo apt-get install -y \
           build-essential \
           checkinstall \
           cmake \
           libasound2-dev \
           libavcodec-dev \
           libavdevice-dev \
           libavfilter-dev \
           libavformat-dev \
           libavutil-dev \
           libcurl4-openssl-dev \
           libfdk-aac-dev \
           libfontconfig-dev \
           libfreetype6-dev \
           libgl1-mesa-dev \
           libjack-jackd2-dev \
           libjansson-dev \
           libluajit-5.1-dev \
           libpulse-dev \
           libqt5x11extras5-dev \
           libspeexdsp-dev \
           libswresample-dev \
           libswscale-dev \
           libudev-dev \
           libv4l-dev \
           libva-dev \
           libvlc-dev \
           libx11-dev \
           libx264-dev \
           libxcb-randr0-dev \
           libxcb-shm0-dev \
           libxcb-xinerama0-dev \
           libxcomposite-dev \
           libxinerama-dev \
           libmbedtls-dev \
           pkg-config \
           python3-dev \
           qtbase5-dev \
           libqt5svg5-dev \
           swig
      - name: 'Restore Chromium Embedded Framework from cache'
        id: cef-cache
        uses: actions/cache@v1
        env:
          CACHE_NAME: 'cef-cache'
        with:
          path: ${{ github.workspace }}/cmbuild/cef_binary_${{ env.CEF_BUILD_VERSION }}_linux64
          key: ${{ runner.os }}-pr-${{ env.CACHE_NAME }}-${{ env.CEF_BUILD_VERSION }}
      - name: 'Install prerequisite: Chromium Embedded Framework'
        if: steps.cef-cache.outputs.cache-hit != 'true'
        shell: bash
        run: |
          curl -kL https://cdn-fastly.obsproject.com/downloads/cef_binary_${{ env.CEF_BUILD_VERSION }}_linux64.tar.bz2 -f --retry 5 -o cef.tar.bz2
          if [ ! -d "${{ github.workspace }}/cmbuild" ]; then mkdir "${{ github.workspace }}/cmbuild"; fi
          tar -C"${{ github.workspace }}/cmbuild" -xjf cef.tar.bz2
      - name: 'Configure'
        shell: bash
        run: |
          mkdir ./build
          cd ./build
          cmake -DUNIX_STRUCTURE=0 -DCMAKE_INSTALL_PREFIX="${{ github.workspace }}/obs-studio-portable" -DBUILD_CAPTIONS=ON -DWITH_RTMPS=ON -DBUILD_BROWSER=ON -DCEF_ROOT_DIR="${{ github.workspace }}/cmbuild/cef_binary_${{ env.CEF_BUILD_VERSION }}_linux64" ..
      - name: 'Build'
        shell: bash
        working-directory: ${{ github.workspace }}/build
        run: make -j4
      - name: 'Package'
        if: success() && (github.event_name != 'pull_request' || contains( github.event.pull_request.labels.*.name, 'Seeking Testers'))
        shell: bash
        run: |
          FILE_DATE=$(date +%Y-%m-%d)
          FILE_NAME=$FILE_DATE-${{ env.OBS_GIT_HASH }}-${{ env.OBS_GIT_TAG }}-linux64.tar.gz
          echo "::set-env name=FILE_NAME::${FILE_NAME}"
          cd ./build
          sudo checkinstall --default --install=no --pkgname=obs-studio --fstrans=yes --backup=no --pkgversion="$(date +%Y%m%d)-git" --deldoc=yes
          mkdir ../nightly
          tar -cvzf "${FILE_NAME}" *.deb
          mv "${FILE_NAME}" ../nightly/
          cd -
      - name: 'Publish'
        if: success() && (github.event_name != 'pull_request' || contains( github.event.pull_request.labels.*.name, 'Seeking Testers'))
        uses: actions/upload-artifact@v2-preview
        with:
          name: '${{ env.FILE_NAME }}'
          path: './nightly/*.tar.gz'
  win64:
    name: 'Windows 64-bit'
    runs-on: [windows-latest]
    env:
      QT_VERSION: '5.10.1'
      CMAKE_GENERATOR: "Visual Studio 16 2019"
      CMAKE_SYSTEM_VERSION: "10.0.18363.657"
      WINDOWS_DEPS_VERSION: '2017'
      VLC_VERSION: '3.0.0-git'
      TWITCH-CLIENTID: ${{ secrets.TWITCH_CLIENTID }}
      TWITCH-HASH: ${{ secrets.TWITCH_HASH }}
      MIXER-CLIENTID: ${{ secrets.MIXER_CLIENTID }}
      MIXER-HASH: ${{ secrets.MIXER_HASH }}
      RESTREAM-CLIENTID: ${{ secrets.RESTREAM-CLIENTID }}
      RESTREAM-HASH: ${{ secrets.RESTREAM-HASH }}
    steps:
      - name: 'Add msbuild to PATH'
        uses: microsoft/setup-msbuild@v1.0.0
      - name: 'Checkout'
        uses: actions/checkout@v2
        with:
          submodules: 'recursive'
      - name: 'Fetch Git Tags'
        shell: bash
        run: |
          git fetch --prune --unshallow
          echo ::set-env name=OBS_GIT_BRANCH::$(git rev-parse --abbrev-ref HEAD)
          echo ::set-env name=OBS_GIT_HASH::$(git rev-parse --short HEAD)
          echo ::set-env name=OBS_GIT_TAG::$(git describe --tags --abbrev=0)
      - name: 'Restore QT dependency from cache'
        id: qt-cache
        uses: actions/cache@v1
        env:
          CACHE_NAME: 'windows-qt-cache'
        with:
          path: ${{ github.workspace }}/cmbuild/QT
          key: ${{ runner.os }}-pr-${{ env.CACHE_NAME }}-${{ env.QT_VERSION }}
      - name: 'Restore pre-built dependencies from cache'
        id: deps-cache
        uses: actions/cache@v1
        env:
          CACHE_NAME: 'windows-deps-cache'
        with:
          path: ${{ github.workspace }}/cmbuild/deps
          key: ${{ runner.os }}-pr-${{ env.CACHE_NAME }}-${{ env.WINDOWS_DEPS_VERSION }}
      - name: 'Restore VLC dependency from cache'
        id: vlc-cache
        uses: actions/cache@v1
        env:
          CACHE_NAME: 'windows-vlc-cache'
        with:
          path: ${{ github.workspace }}/cmbuild/vlc
          key: ${{ runner.os }}-pr-${{ env.CACHE_NAME }}-${{ env.VLC_VERSION }}
      - name: 'Restore CEF dependency from cache (64 bit)'
        id: cef-cache
        uses: actions/cache@v1
        env:
          CACHE_NAME: 'windows-cef-64-cache'
        with:
          path: ${{ github.workspace }}/cmdbuild/cef_binary_${{ env.CEF_BUILD_VERSION }}_windows64_minimal
          key: ${{ runner.os }}-pr-${{ env.CACHE_NAME }}-${{ env.CEF_BUILD_VERSION }}
      - name: 'Install prerequisite: QT'
        if: steps.qt-cache.outputs.cache-hit != 'true'
        run: |
          curl -kLO https://cdn-fastly.obsproject.com/downloads/Qt_${{ env.QT_VERSION }}.7z -f --retry 5 -C -
          7z x Qt_${{ env.QT_VERSION }}.7z -o"${{ github.workspace }}/cmbuild/QT"
      - name: 'Install prerequisite: Pre-built dependencies'
        if: steps.deps-cache.outputs.cache-hit != 'true'
        run: |
          curl -kLO https://cdn-fastly.obsproject.com/downloads/dependencies${{ env.WINDOWS_DEPS_VERSION }}.zip -f --retry 5 -C -
          7z x dependencies${{ env.WINDOWS_DEPS_VERSION }}.zip -o"${{ github.workspace }}/cmbuild/deps"
      - name: 'Install prerequisite: VLC'
        if: steps.vlc-cache.outputs.cache-hit != 'true'
        run: |
          curl -kL https://cdn-fastly.obsproject.com/downloads/vlc.zip -f --retry 5 -o vlc.zip
          7z x vlc.zip -o"${{ github.workspace }}/cmbuild/vlc"
      - name: 'Install prerequisite: Chromium Embedded Framework'
        if: steps.cef-cache.outputs.cache-hit != 'true'
        run: |
          curl -kL https://cdn-fastly.obsproject.com/downloads/cef_binary_${{ env.CEF_VERSION }}_windows64_minimal.zip -f --retry 5 -o cef.zip
          7z x cef.zip -o"${{ github.workspace }}/cmbuild"
      - name: 'Configure'
        run: |
          mkdir ./build
          mkdir ./build64
          cd ./build64
          cmake -G"${{ env.CMAKE_GENERATOR }}" -A"x64" -DCMAKE_SYSTEM_VERSION="${{ env.CMAKE_SYSTEM_VERSION }}" -DBUILD_BROWSER=true -DBUILD_CAPTIONS=true -DCOMPILE_D3D12_HOOK=true -DDepsPath="${{ github.workspace }}/cmbuild/deps/win64" -DQTDIR="${{ github.workspace }}/cmbuild/QT/${{ env.QT_VERSION }}/msvc2017_64" -DCEF_ROOT_DIR="${{ github.workspace }}/cmdbuild/cef_binary_${{ env.CEF_VERSION }}_windows64_minimal" -DCOPIED_DEPENDENCIES=FALSE -DCOPY_DEPENDENCIES=TRUE ..
      - name: 'Build'
        run: msbuild /m /p:Configuration=RelWithDebInfo .\build64\obs-studio.sln
      - name: 'Package'
        if: success() && (github.event_name != 'pull_request' || contains( github.event.pull_request.labels.*.name, 'Seeking Testers'))
        run: |
          $env:FILE_DATE=(Get-Date -UFormat "%F")
          $env:FILE_NAME="${env:FILE_DATE}-${{ env.OBS_GIT_HASH }}-${{ env.OBS_GIT_TAG }}-win64.zip"
          echo "::set-env name=FILE_NAME::${env:FILE_NAME}"
          robocopy .\build64\rundir\RelWithDebInfo .\build\ /E /XF .gitignore
          7z a ${env:FILE_NAME} .\build\*
      - name: 'Publish'
        if: success() && (github.event_name != 'pull_request' || contains( github.event.pull_request.labels.*.name, 'Seeking Testers'))
        uses: actions/upload-artifact@v2-preview
        with:
          name: '${{ env.FILE_NAME }}'
          path: '*-win64.zip'

  win32:
    name: 'Windows 32-bit'
    runs-on: [windows-latest]
    env:
      QT_VERSION: '5.10.1'
      CMAKE_GENERATOR: "Visual Studio 16 2019"
      CMAKE_SYSTEM_VERSION: "10.0.18363.657"
      WINDOWS_DEPS_VERSION: '2017'
      TWITCH-CLIENTID: ${{ secrets.TWITCH_CLIENTID }}
      TWITCH-HASH: ${{ secrets.TWITCH_HASH }}
      MIXER-CLIENTID: ${{ secrets.MIXER_CLIENTID }}
      MIXER-HASH: ${{ secrets.MIXER_HASH }}
      RESTREAM-CLIENTID: ${{ secrets.RESTREAM-CLIENTID }}
      RESTREAM-HASH: ${{ secrets.RESTREAM-HASH }}
    steps:
      - name: 'Add msbuild to PATH'
        uses: microsoft/setup-msbuild@v1.0.0
      - name: 'Checkout'
        uses: actions/checkout@v2
        with:
          submodules: 'recursive'
      - name: 'Fetch Git Tags'
        shell: bash
        run: |
          git fetch --prune --unshallow
          echo ::set-env name=OBS_GIT_BRANCH::$(git rev-parse --abbrev-ref HEAD)
          echo ::set-env name=OBS_GIT_HASH::$(git rev-parse --short HEAD)
          echo ::set-env name=OBS_GIT_TAG::$(git describe --tags --abbrev=0)
      - name: 'Restore QT dependency from cache'
        id: qt-cache
        uses: actions/cache@v1
        env:
          CACHE_NAME: 'qt-cache'
        with:
          path: ${{ github.workspace }}/cmbuild/QT
          key: ${{ runner.os }}-pr-${{ env.CACHE_NAME }}-${{ env.QT_VERSION }}
      - name: 'Restore pre-built dependencies from cache'
        id: deps-cache
        uses: actions/cache@v1
        env:
          CACHE_NAME: 'deps-cache'
        with:
          path: ${{ github.workspace }}/cmbuild/deps
          key: ${{ runner.os }}-pr-${{ env.CACHE_NAME }}-${{ env.WINDOWS_DEPS_VERSION }}
      - name: 'Restore VLC dependency from cache'
        id: vlc-cache
        uses: actions/cache@v1
        env:
          CACHE_NAME: 'vlc-cache'
        with:
          path: ${{ github.workspace }}/cmbuild/vlc
          key: ${{ runner.os }}-pr-${{ env.CACHE_NAME }}-${{ env.WINDOWS_VLC_VERSION }}
      - name: 'Restore CEF dependency from cache (32 bit)'
        id: cef-cache
        uses: actions/cache@v1
        env:
          CACHE_NAME: 'cef-32-cache'
        with:
          path: ${{ github.workspace }}/cmdbuild/cef_binary_${{ env.CEF_BUILD_VERSION }}_windows32_minimal
          key: ${{ runner.os }}-pr-${{ env.CACHE_NAME }}-${{ env.CEF_VERSION }}
      - name: 'Install prerequisite: QT'
        if: steps.qt-cache.outputs.cache-hit != 'true'
        run: |
          curl -kLO https://cdn-fastly.obsproject.com/downloads/Qt_${{ env.QT_VERSION }}.7z -f --retry 5 -C -
          7z x Qt_${{ env.QT_VERSION }}.7z -o"${{ github.workspace }}/cmbuild/QT"
      - name: 'Install prerequisite: Pre-built dependencies'
        if: steps.deps-cache.outputs.cache-hit != 'true'
        run: |
          curl -kLO https://cdn-fastly.obsproject.com/downloads/dependencies${{ env.WINDOWS_DEPS_VERSION }}.zip -f --retry 5 -C -
          7z x dependencies${{ env.WINDOWS_DEPS_VERSION }}.zip -o"${{ github.workspace }}/cmbuild/deps"
      - name: 'Install prerequisite: VLC'
        if: steps.vlc-cache.outputs.cache-hit != 'true'
        run: |
          curl -kL https://cdn-fastly.obsproject.com/downloads/vlc.zip -f --retry 5 -o vlc.zip
          7z x vlc.zip -o"${{ github.workspace }}/cmbuild/vlc"
      - name: 'Install prerequisite: Chromium Embedded Framework'
        if: steps.cef-cache.outputs.cache-hit != 'true'
        run: |
          curl -kL https://cdn-fastly.obsproject.com/downloads/cef_binary_${{ env.CEF_VERSION }}_windows32_minimal.zip -f --retry 5 -o cef.zip
          7z x cef.zip -o"${{ github.workspace }}/cmbuild"
      - name: 'Configure'
        run: |
          mkdir ./build
          mkdir ./build32
          cd ./build32
          cmake -G"${{ env.CMAKE_GENERATOR }}" -A"Win32" -DCMAKE_SYSTEM_VERSION="${{ env.CMAKE_SYSTEM_VERSION }}" -DBUILD_BROWSER=true -DBUILD_CAPTIONS=true -DCOMPILE_D3D12_HOOK=true -DDepsPath="${{ github.workspace }}/cmbuild/deps/win32" -DQTDIR="${{ github.workspace }}/cmbuild/QT/${{ env.QT_VERSION }}/msvc2017" -DCEF_ROOT_DIR="${{ github.workspace }}/cmdbuild/cef_binary_${{ env.CEF_VERSION }}_windows32_minimal" -DCOPIED_DEPENDENCIES=FALSE -DCOPY_DEPENDENCIES=TRUE ..
      - name: 'Build'
        run: msbuild /m /p:Configuration=RelWithDebInfo .\build32\obs-studio.sln
      - name: 'Package'
        if: success() && (github.event_name != 'pull_request' || contains( github.event.pull_request.labels.*.name, 'Seeking Testers'))
        run: |
          $env:FILE_DATE=(Get-Date -UFormat "%F")
          $env:FILE_NAME="${env:FILE_DATE}-${{ env.OBS_GIT_HASH }}-${{ env.OBS_GIT_TAG }}-win32.zip"
          echo "::set-env name=FILE_NAME::${env:FILE_NAME}"
          robocopy .\build32\rundir\RelWithDebInfo .\build\ /E /XF .gitignore
          7z a ${env:FILE_NAME} .\build\*
      - name: 'Publish'
        if: success() && (github.event_name != 'pull_request' || contains( github.event.pull_request.labels.*.name, 'Seeking Testers'))
        uses: actions/upload-artifact@v2-preview
        with:
          name: '${{ env.FILE_NAME }}'
          path: '*-win32.zip'

#binaries
*.exe
*.dll
*.dylib
*.so

#cmake
/cmbuild/
/build/
/build32/
/build64/
/release/
/release32/
/release64/
/debug/
/debug32/
/debug64/
/builds/
.vs/
*.o.d
*.ninja
.ninja*
.dirstamp

#xcode
*.xcodeproj/

#clion
.idea/

#other stuff (windows stuff, qt moc stuff, etc)
Release_MD/
Release/
Debug/
x64/
ipch/
GeneratedFiles/
.moc/
/UI/obs.rc
.vscode/

/other/

#make stuff
configure
depcomp
install-sh
Makefile.in
Makefile

#python
__pycache__

#sphinx
/docs/sphinx/_build/*
!/docs/sphinx/_build/.gitignore
!/docs/sphinx/Makefile

#random useless file stuff
*.dmg
*.app
.DS_Store
.directory
.hg
.depend
tags
*.trace
*.vsp
*.psess
*.swp
*.dat
*.clbin
*.log
*.tlog
*.sdf
*.opensdf
*.xml
*.ipch
*.css
*.xslt
*.aps
*.suo
*.ncb
*.user
*.lo
*.ilk
*.la
*.o
*.obj
*.pdb
*.res
*.dep
*.zip
*.lnk
*.chm
*~
.DS_Store
*/.DS_Store
*/**/.DS_Store

[submodule "plugins/win-dshow/libdshowcapture"]
	path = plugins/win-dshow/libdshowcapture
	url = https://github.com/obsproject/libdshowcapture.git
[submodule "plugins/mac-syphon/syphon-framework"]
	path = plugins/mac-syphon/syphon-framework
	url = https://github.com/palana/Syphon-Framework.git
[submodule "plugins/enc-amf"]
	path = plugins/enc-amf
	url = https://github.com/obsproject/obs-amd-encoder.git
[submodule "plugins/obs-browser"]
	path = plugins/obs-browser
	url = https://github.com/obsproject/obs-browser.git
[submodule "plugins/obs-vst"]
	path = plugins/obs-vst
	url = https://github.com/obsproject/obs-vst.git
[submodule "plugins/obs-outputs/ftl-sdk"]
	path = plugins/obs-outputs/ftl-sdk
	url = https://github.com/Mixer/ftl-sdk.git

Jim <obs.jim@gmail.com>
Zachary Lund <admin@computerquip.com>
Benjamin Klettbach <b.klettbach@gmail.com>
BtbN <btbn@btbn.de>
John Bradley <jrb@turrettech.com>
HomeWorld <homeworld@gmail.com> hwdro <pdarvaru@yahoo.com>
Michael Fabian Dirks <info@xaymar.com> <michael.dirks@xaymar.com>
Martell Malone <martellmalone@gmail.com>

Original Author: Hugh Bailey ("Jim")

Contributors are sorted by their amount of commits / translated strings.

Contributors:
Fabian5701
Jim
Palana
fryshorts
R1CH
BtbN
DDRBoxman
Gol-D-Ace
Shaolin
kc5nra
Ryan Foster
Zachary Lund
cg2121
Xaymar
SuslikV
Rodney
Christoph Hohmann
pkv
Martell Malone
HomeWorld
dodgepong
Alex Anderson
juvester
Dmitry-Me
Fenrir
Radzaquiel
Socapex
Cephas Reis
Dillon Pentz
Kurt Kartaltepe
Luke Yelavich
mntone
sorayuki
Andrew Surzhynskyi
Skyler Lipthay
Arkkis
Danni
GoaLitiuM
Hunter L. Allen
Marvin Scholz
Michel Snippe
Quinn Damerell
Tjienta Vara
craftwar
Clayton Groeneveld
Ilya Melamed
Jess Mayo
Jimi Huotari
Kris Moore
Alexandre Vicenzi
Carl Fürstenberg
Kilian von Pflugk
Anry
CoDEmanX
H4ndy
Ján Mlynek
nleseul
Benjamin Klettbach
Bird, Christopher
Bl00drav3n
Blackhive
Charles Ray Shisler III
Daniel Lopez
David Cooper
Exeldro
Jeremiah Senkpiel
Joseph El-Khouri
Matt WizardCM
Palakis
Philip Haynes
Robin Hielscher
Serge Paquet
Timo Gurr
bl
shiina424
shousa
vokama
Alexander Schittler
Andreas Reischuck
Andrei Nistor
Azat Khasanshin
Brian S. Stephan
Chaturbate
Dead133
DungFu
Eric Bataille
Igor Bochkariov
Justin Love
Lexsus
Lionheart Zhang
Lucian Poston
Matt Morrissette
Matthew McNamara
Michael Goulet
Skid-Inc
Taylor Blau
Tristan Matthews
Warchamp7
Warren Turkal
adray
bootkiller
e00E
jpk
paibox
skwerlman
taesheren
yogpstop
Aarni Koskela
Abelardo E. Mendoza
Aesen Vismea
Aidan Epstein
Akagi201
Alcaros
Alexander Uhlmann
Alexandre Biny
Alexandre Paré
Ali Kaviani
Anthony Catel
Anthony Super
Artem Merkulov
Asgeir Mortensen
Autumin
Aydin Akan
Bas van Meel
Bazhenoff
Beau Russell
Ben Krueger
Ben Stahl
Benjamin Schubert
Bernd Buschinski
Bjorn
Bongalive
C. Daniel Sanchez R
Caitlin Potter
Caleb Anderson
CallumHoward
Cam
Can Bal
Cheeseness
CommanderRoot
Copy Liu
Cray Elliott
Dan Dascalescu
David McMackins II
Derrick Lambert
Dmitry Odintsov
Emil Sayahi
EpicCoder
Ethan Lee
Fazrad Karkhani
Frank Gehann
Fred Emmott
Garrett
Giorgio Pellero
Gregory Luneau
Grigorii Chirkov
Guillermo A. Amaral
Gökberk Yaltıraklı
Haden F
Han-Tai Chen
Hicham LEMGHARI
Iblis Lin
J Lynn
J.D. Purcell
Jake Probst
James Hurley
Jamy Timmermans
JetMeta
Jimmy Berry
Jkoan
John Cheng
Jonathan
Joshua Rowe
Julian Miller
Kazuki Oishi
Kevin
Kevin Tardif
Lasse Dalegaard
Lioncash
Lukas Monka
Makeenon
Marc Chambers
Mark Vaughn
Mathias Panzenböck
Matthew Szatmary
MedicMomcilo
Micah Elizabeth Scott
Michael Hoang
Momcilo Medic
Murnux
Nicolas F
Night
Olivier Humbert
Olle Kelderman
Patrick Ancillotti
Peter SZTANOJEV
Philip Loche
Ricardo Constantino
Rodrigo Ipince
Ryan Sullivan
SammyJames
Serge Martial Nguetta
Seth Murphy
Seung-Woo Kim
Shamun
Simon
Sophie Hamilton
SoraYuki
Steve Wills
Tatsuyuki Ishi
Taylor
Teemu Kauhanen
Terje Gundersen
Thomas Crider
Thomas De Schampheleire
Thomas McGrew
Thomas Schnitzler
Tom Peeters
Tom Pichler
TotalCaesar659
Wahaj Dar
Weikardzaena
Will Jamieson
William Casarin
Wouter
Younes SERRAJ
Yvo
Ziemas
adocilesloth
astudios123
bla
bluekirby0
boombatower
bth
chinasarft
comex
cryptonaut
dennis
eastkiki
fefine
jamacanbacn
jeremiah
ka’imi
lemmi
mape
mhabedinpour
michael bishop
nd
notmark
pantonvich
partouf
pipll
pkuznetsov
probonopd
raincomplex
repeat
rkfg
rpslack
sam8641
thekrzos
trwnh
vic
vividnightmare
wangrui05
wayne wang

Translators:
Translators:
 Afrikaans:
  - Samuel Nthoroane (Samuel_Nthoroane)
  - speedytechdev
  - Gol D. Ace (goldace)
 Albanian:
  - Aredio Vani (aredio.vani)
  - Albin Pllana (albinnpllanaa)
  - Gol D. Ace (goldace)
 Arabic:
  - ZILZAL
  - Abdullah AL-Qahtani (Za7ef_SA)
  - majdcomp
  - aaakjt
  - معتصم دعنا (rozana-media)
  - Gol D. Ace (goldace)
  - REMOVED_USER
  - Saleh oukiki (salehoukiki)
  - Vainock
  - Tensai
  - Mustafa2018
  - BeleganStartup (BeleganStartup)
  - Ndalabo Taema (hake_bsowq)
  - Hani Sweileh (hno.sweileh)
  - BWU Wheelman (Wheelman)
  - FC Barcelona HD (kurdnews)
  - Yhea Ahmad (yhea.ahmad7741)
  - Ahmed Hawam (Hawam)
  - Mnsor The-Ghost (mnsor1722011)
  - dodgepong
  - chaironeko
  - Hadi Gamer (hadigamer3131)
 Azerbaijani:
  - Shahin Farzaliyev (Khan27)
 Bashkir:
  - Churikov Radmir Sergeevich (RadMir2)
 Basque:
  - Alexander Gabilondo (alexgabi)
  - Xabier Aramendi (azpidatziak)
  - REMOVED_USER
  - Osoitz (Osoitz)
  - Gol D. Ace (goldace)
  - Vainock
  - EG Gamer (eggamer131)
  - REMOVED_USER
  - txaro (txaro)
  - etxondoko
  - dodgepong
  - jolaus
  - ziztazale
  - alaitzgabi
  - zientek
 Bengali:
  - shamuntohamd
  - Rudro Rasel (rdrrsl)
  - REMOVED_USER
  - Vainock
  - Nurul Huda (nurulhuda859)
  - Gol D. Ace (goldace)
  - Al Arafat (zonoiko)
 Bulgarian:
  - j3dy
  - Dremski
  - Zakxaev68
  - kalmarin
  - Seyhan Halil (yildirim17)
  - Martin Georgiev (DivideByNone)
  - Vainock
  - REMOVED_USER
  - Viktor Kitov (viktorkitov)
  - Stoyan Stoyanov (sstoyanov)
  - Stanislav_Evtimov
  - Gol D. Ace (goldace)
  - Ivan (SKDown)
  - djradimix
  - Warchamp7
  - dodgepong
 Catalan:
  - Jaime Muñoz Martín (jmmartin_5)
  - Benet R. i Camps (BennyBeat) (BennyBeat)
  - jmontane
  - Nil Campamà (Soifam)
  - Aleix Vidal i Gaya (leixet)
  - Vainock
  - REMOVED_USER
  - Gol D. Ace (goldace)
  - Pere O. (gotrunks)
  - Aniol Pagès (aniolpages)
  - REMOVED_USER
  - dodgepong
  - REMOVED_USER
 Chinese Simplified:
  - Bob Liu (Akagi201)
  - AlexGuo1998 (AlexGuo1998)
  - wwj402_github
  - Forbidden (Origami)
  - Hexcolyte (Hexcolyte)
  - AthlonHD (AthlonHD)
  - Gol D. Ace (goldace)
  - Licardo (Licardo)
  - PabloLiu (719018105)
  - REMOVED_USER
  - fangzheng (fangzheng)
  - Sasasu (Sasasu)
  - David Kuo (s50407s)
  - Vainock
  - REMOVED_USER
  - 鲜童 (xiananjyzy)
  - Rudro Rasel (rdrrsl)
  - cai_miao
  - Opportunity (OpportunityLiu)
  - MarsYoung
  - Inku Xuan (inkuxuan)
  - SR_Mango
  - Yi-Jyun Pan (pan93412)
  - yunluzhang
  - copyliu
  - Boyuan Yang (073plan)
  - Richard Stanway (r1ch)
  - Bing Feng (fengbing123)
  - 科技小白堂 (lipeng0820)
  - WeiYuanStudio
  - WaterOtaku
  - sorayuki
  - REMOVED_USER
  - dodgepong
  - Bob Wei (BobWaver)
  - OYYZ
  - FaZe Fakay (fazefakay)
  - wordlessWind (wordlesswind)
  - cylin
  - Jiaxun Yang (flygoatfree)
  - 赵杭灵 (h1679083640)
  - Martinzzz
  - defeng
  - REMOVED_USER
 Chinese Traditional:
  - TzeKei Lee (chikei)
  - Yi-Jyun Pan (pan93412)
  - Florid (u900011)
  - Julian_Lai
  - Michael Yeh (hinet60613)
  - dodgepong
  - myjourney in Steemit (myjourney)
  - You-Ruei Tzeng (e222et)
  - open3 (kanbi99999)
  - David Kuo (s50407s)
  - Gol D. Ace (goldace)
  - Vainock
  - louis921222
  - craftwar
  - Thomas (thomassth)
  - REMOVED_USER
  - xixiaofan (Hs0)
  - Watson Tsai (ashaneba)
  - Han-Jen Cheng (notexist)
  - Inndy.Lin (inndy)
  - ak-47root
  - abc0922001
  - Meng Hao Li (GazCore)
  - Append Huang (append)
  - fangzheng (fangzheng)
  - zhtw
  - JackYeah
  - Rudro Rasel (rdrrsl)
  - cai_miao
  - REMOVED_USER
  - Jimmy Huang (f56112000)
  - 曹恩逢 (nelson22768384)
  - FaZe Fakay (fazefakay)
  - tomoe-musashi
  - chaironeko
  - WeingHong
  - Bob Wei (BobWaver)
  - Tatsujin (c910335)
  - watadon
  - REMOVED_USER
 Croatian:
  - medicmomcilo
  - srdjan_m
  - Runicar (dajtisina)
  - Wildrage
  - OfficialwobY
  - Gol D. Ace (goldace)
  - Maky (the.real.maky)
  - Vainock
  - REMOVED_USER
  - dodgepong
 Czech:
  - Jirka 'Venty' Michel (VentyCZ)
  - Kryštof Černý (cleverline1mc)
  - Vainock
  - Gol D. Ace (goldace)
  - JIMMY (vopasek)
  - Kiznoh
  - Sawanyo
  - Erik Bročko (ericek111)
  - Daniel Padrta (dpadrta)
  - dodgepong
  - REMOVED_USER
 Danish:
  - NCAA (NCAA)
  - Jens Hyllegaard (Hyllegaard)
  - scootergrisen
  - MaltahlGaming (maltahlgaming)
  - Anders G. Jørgensen (spirit55555)
  - Gol D. Ace (goldace)
  - Vainock
  - Anders Urban (minikaliffen)
  - Richard Stanway (r1ch)
  - Marque Ziqulr (lugtelort)
  - REMOVED_USER
  - Johan Keller Jensen (JKeller)
  - HeroGamers (Fido2603)
  - Christian Henriksen (cnhenriksen)
  - Nicolai Skødt Holmgaard (Nicolai9852)
  - Daniel Aundal (aundal)
  - dodgepong
 Dutch:
  - Eric Bataille (ThoNohT)
  - Michel Snippe (michelsnippe)
  - Greendweller
  - Albakham (albakham)
  - exeldro (exeldro)
  - Coen (Trigstur)
  - Danny (Dkamps18)
  - Nicole (Dutchess_Nicole)
  - robbert0891 (robbertoorschot38)
  - Gol D. Ace (goldace)
  - Kjetil Verstrepen (kjetilv)
  - Harm van den Hoek (harm27)
  - Jasper J (JassieJ)
  - Vainock
  - Lars van Wijk (lhjvanwijk98)
  - Bond-009
  - markpc
  - Jennifer Falco (JenTheBluePanda)
  - andymidside
  - F_Producktions
  - b__dm
  - Bo Alsemgeest (bo.alsemgeest.wausie)
  - lokerhp
  - REMOVED_USER
  - JorRy
  - Julian Meijboom (julianmeijboom)
  - RyyzQ
  - JustMaffie (JustMaffie)
  - Rik Roukens (rikroukens)
  - dodgepong
 Estonian:
  - MartinEwing
  - AndresTraks
  - REMOVED_USER
  - C3kos (C3kos1)
  - Gol D. Ace (goldace)
  - Vainock
 Filipino:
  - dwaeji-aizelle (dwaeji-aizelle)
  - nyakayed (nyakayed)
  - Loyd Stephen Jayme (loydjayme25)
  - jermel
  - Gol D. Ace (goldace)
  - Vainock
  - REMOVED_USER
  - Raylir
  - vinzruzell
 Finnish:
  - Arkkis (j)
  - Jarska
  - dodgepong
  - Tero Keso (tero.keso)
  - Gol D. Ace (goldace)
  - Pyscowicz (Pyscowicz)
  - Vainock
  - REMOVED_USER
  - Obama (Obama44)
  - Ville Närhi (daimaah)
  - Arttu Ylhävuori (arttu.ylhavuori)
  - chaironeko
 French:
  - pkviet
  - radzaquiel
  - Stéphane Lepin (Palakis)
  - Tocram2 (tocram2)
  - REMOVED_USER
  - Benjamin Cambour (lesinfox)
  - Yberion
  - Léo (leeo97one)
  - RisedSky (THEMINECRAFT951)
  - Pikana (Pikana)
  - BoboopTeam
  - DoK_-
  - Guillaume Turchini (orion78fr)
  - DarckCrystale
  - Ben Turner (ben-turner)
  - DarkInFire
  - Theguiguix
  - Gol D. Ace (goldace)
  - Youtubeur FR│Giaco35 (Giaco35)
  - Vainock
  - steve_fr
  - kyllian (tardigradeus)
  - Deski_ (Deski_)
  - GANGAT Naeem (zboggum)
  - REMOVED_USER
  - Aime23
  - Grisou2907
  - BaguetteDePain_
  - illusdidi
  - Mathieu Hautebas (matteyeux)
  - 🌠 DarK | #Hello 🌠 (DarKTV_FR)
  - Yolopix
  - Gabriel Dugny (Gabigabigo)
  - tburette
  - Adrien “GameZone Tv” de Decker (redcraft007)
  - Richard Stanway (r1ch)
  - Zalki
  - McGuygnol
  - SkytAsul (skytasul)
  - Alexis Brandner (Alexinfos)
  - Camille Nury (kamsdu30)
  - dodgepong
  - Albakham (albakham)
  - Warchamp7
  - SkylixX
  - Jean-Mathieu Jm Samson (sjm450666)
  - chaironeko
  - William Dunn (williamdunnperso)
  - HoPollo (HoPolloTV)
  - Antwan
 Galician:
  - mbouzada
  - Xesús M. Mosquera Carregal (xesusmosquera)
  - Iváns (Ivans_translator)
  - Vainock
  - REMOVED_USER
  - Gol D. Ace (goldace)
  - chaironeko
  - dodgepong
  - Máximo A. Coejo (mcoejo)
 Georgian:
  - georgianizator
  - Vainock
  - EduCare Razmik Badalyan (badalyanrazmik)
  - REMOVED_USER
  - Gol D. Ace (goldace)
 German:
  - Gol D. Ace (goldace)
  - Vainock
  - Michael Fabian Dirks (Xaymar)
  - Sven Kirschbaum (fallobst22) (fallobst22)
  - Gregor Bigalke (gregtcltk)
  - dodgepong
  - Manuel (ElectronicWar)
  - Dennis Giebert (Isegrim) (isegrimderwolf)
  - Tim (robske_110) (robske110)
  - Marvin J. (juettemarvin)
  - mdod
  - Benjamin Klettbach (benklett)
  - Jonas Otto (jottosmail)
  - REMOVED_USER
  - WurstOnAir
  - lebaston100.de
  - REMOVED_USER
  - Richard Stanway (r1ch)
  - Palana
  - Jonathan (macburgerjunior)
  - Enderdrache LP (enderdrachelp)
  - Prince_of_Raop
  - Robin Hielscher (Jack0r)
  - Patrick Frings (Ragnos)
  - Lord Aidan (BadSideofBright)
  - REMOVED_USER
  - AndreLeonardo (andreleonardoyt)
  - BoJustus
  - REMOVED_USER
  - REMOVED_USER
  - Hadi Gamer (hadigamer3131)
  - Tiim
  - Fabian Küntzler (tringlp)
  - Lasse Niermann (TheLasseHD)
  - Fruxh#3282 (Fruxh)
  - Glocker | Felix L. (Haard22)
  - Dominik Weissenseel (domisum)
  - Elementies
  - CisBetter
  - Martina Rauscheder (pixelpuppe)
  - korte.jan (jkoankerich)
  - Enderdrache LP (michaelroth882)
  - Laurenz (El3ctroGh0stL)
  - Reboot (reboot)
  - Kilian von Pflugk (kilianvonpflugk)
  - Poolitzer
 Greek:
  - REMOVED_USER
  - Katerina (katerinaramm)
  - Scourgemcdak
  - Mepharees
  - Tasos Sahanidis (tatokis)
  - George T. (tzikas97)
  - Vainock
  - Alex Kalles (alexakis1997)
  - Warmaster
  - Gol D. Ace (goldace)
  - iosifidis
  - Giorgos (giorgospap206)
  - Themis T. (Deminho)
  - REMOVED_USER
  - dodgepong
  - chaironeko
  - Dimitris P. (dimitrisp2)
 Hebrew:
  - amirsher
  - lonelywolf11
  - djsavta
  - Vainock
  - Chemi (Chemi)
  - Gol D. Ace (goldace)
  - REMOVED_USER
  - Light1c3
  - epic_ziver_D (epic_ziver_D)
  - REMOVED_USER
  - TheOver (upmeboost)
  - yair (5shekel)
  - אפיק רוזנר (afikr333)
  - ghsi
  - Tal Machani (talmachani)
 Hindi:
  - Saswata Banerjee (azure.saswata)
  - Rahul Dhangar (rahuldhangar)
  - SneakyFish5
  - shamuntohamd
  - Gol D. Ace (goldace)
 Hungarian:
  - Gige
  - Balázs Meskó (mesko.balazs)
  - Gol D. Ace (goldace)
  - Vainock
  - REMOVED_USER
  - Balázs Meskó (meskobalazs)
  - Adam Liszkai (adamos42)
  - vargag159
  - abydosan (abydoshun)
  - dodgepong
  - Sigge Stjärnholm (Kladdy)
 Italian:
  - Manfre#9262 (manfre)
  - LordShadow95
  - Marocco2
  - imcesca
  - Tiwi90 (tiwi90)
  - smart2128
  - Sergio Beneduce (sbeneduce)
  - Michele (ScrappyCocco)
  - Albakham (albakham)
  - dodgepong
  - Alessandro Sarto (alesarto03)
  - Tommaso Cammelli (tomganguz) (tomganguz)
  - mauriziopersi
  - Martazza (Martazza)
  - ScemEnzo
  - Edoardo Macrì (edomacri)
  - Vainock
  - riccardotornesello
  - Gol D. Ace (goldace)
  - Edoardo “OfficialDJMela” Macrì (agersforum)
  - Silpheel
  - gianni morandi (strabbioboy)
  - REMOVED_USER
  - Andrea-M3
  - Fisherozzo
  - DomenicoDD (DomenicoDD)
  - SkyLion
  - Federico Tensi (habby1337)
  - NightMat (NightMat)
  - Alessandro Iepure (alessandro_iepure)
  - Daniele02_777
  - Valerio Ragni (ragni.valerio)
  - Alessandro Levante (lallo2401)
  - Umberto Croci (umbertolb)
 Japanese:
  - shousa
  - Kenta Takumi (kenta0644)
  - Bhanu Victor DiCara (SplatRT)
  - dodgepong
  - 森の子リスのミーコの大冒険 (Phroneris)
  - CKK COBALT (CKKCOBALT)
  - Vainock
  - Gol D. Ace (goldace)
  - ato lash (hal_shu_sato)
  - REMOVED_USER
  - 神成フィルム (kami00nari)
  - Bob Liu (Akagi201)
  - Alex Shafer (enzanki-ars)
  - Yuki Yu (Yukiyu)
  - chaironeko
  - TH_238
 Khmer:
  - ប៉ុកណូ រ៉ូយ៉ាល់ (poknoroyal68)
  - បងមាន តែអូន (cheaiphone267)
  - Gol D. Ace (goldace)
 Korean:
  - ynetwork (ynetwork)
  - Vainock
  - Gol D. Ace (goldace)
  - RedditRook
  - Jong Kwon Choi (dailypro)
  - Ra.Workspace (RaWouk)
  - REMOVED_USER
  - Russell (crimeroyal)
  - REMOVED_USER
  - Charles Wallis (charlestw127)
  - antome
  - 김동현 (ehehguu)
  - dodgepong
  - REMOVED_USER
 Kurdish:
  - Omer Kurdish (OmerKurd)
  - Vainock
 Lithuanian:
  - Justas Vilimas (tyntas)
  - xNaii (lyrikas5)
  - REMOVED_USER
  - Vainock
  - Gol D. Ace (goldace)
 Malay:
  - amsyar ZeRo (amsyarminer555)
  - WeingHong
  - REMOVED_USER
  - Gol D. Ace (goldace)
  - Vainock
  - dodgepong
 Mongolian:
  - begjan
  - Bilguun Ochirbat (Bil0203)
  - REMOVED_USER
  - Vainock
  - Gol D. Ace (goldace)
 Norwegian Bokmal:
  - Imre Kristoffer Eilertsen (DandelionSprout)
  - Taesh (magnusmbratteng)
  - Patrick Williamson (wpatrick59)
  - Oddbjørn Grytdal (Fooshi)
  - Mats Edvin Aarø (matsedvin)
  - dodgepong
  - LandyLERThERmfLOpi
  - areedw
  - mgKaiztra
  - Tommy (nwgat)
  - Syver Stensholt (sssandum)
  - Alex Thomassen (Decicus)
  - Vainock
  - Sander Skjegstad (r530er)
  - Gol D. Ace (goldace)
  - LBlend
  - OsteHovel
  - REMOVED_USER
  - Legend27
  - REMOVED_USER
  - chaironeko
  - Mats Andreassen (MatsA)
 Norwegian Nynorsk:
  - Imre Kristoffer Eilertsen (DandelionSprout)
  - Eiliv
  - morden
  - LandyLERThERmfLOpi
  - REMOVED_USER
  - Gol D. Ace (goldace)
 Persian:
  - koper
  - MZ MAXIMUM (mahdigamermax)
  - Alireza Firouzi (pikhoshorg)
  - peymanr34
  - Vainock
  - Gol D. Ace (goldace)
  - Berrely
  - Navid Sadeghieh (nsadeghieh)
  - Amirhossein yousefi (amir.sara)
 Pirate English:
  - Matt Gajownik (WizardCM)
  - Matthew Hatcher (MatthewSH)
  - Uaiquqjwnsns
  - Isabel L (william__)
  - REMOVED_USER
  - jkcoaster
  - Vainock
  - Coen (Trigstur)
  - Charlie W. (wallichc)
  - Emu-Phoenix
  - Gol D. Ace (goldace)
  - iltrof
  - Berrely
  - chaironeko
  - ncb
  - MaltahlGaming (maltahlgaming)
  - Mepharees
  - Redback93 (Redback93)
  - REMOVED_USER
  - kkartaltepe
  - REMOVED_USER
 Polish:
  - Tomasz 'grocal' Grodzki (grocal)
  - Albakham (albakham)
  - Vassamo (jotpl69)
  - The Syntox (TheSyntox)
  - Michał Durak (micechal)
  - Damian Korcz (damikiller)
  - Łukasz Wójcik (lwojcik)
  - Vainock
  - Daniel Wieczorek (Kennyluz)
  - sebek1pan
  - Mateusz (Silesianek)
  - Gol D. Ace (goldace)
  - REMOVED_USER
  - opl
  - popek069
  - Piteriuz
  - Julia Drewniak (ewagsi)
  - dodgepong
  - Michał Lewczak (michal200507)
  - Patryk Kunda (ner.i.ol)
  - REMOVED_USER
  - Kubix1
  - REMOVED_USER
 Portuguese:
  - Ev1lbl0w
  - REMOVED_USER
  - Manuela Silva (mansil)
  - joaofvieira
  - André Biscaia (LazP)
  - Tomás Antunes (tomasantunes)
  - Albakham (albakham)
  - dodgepong
  - joaoboia
  - alexandre433
  - Vainock
  - REMOVED_USER
  - REMOVED_USER
  - Gol D. Ace (goldace)
  - REMOVED_USER
  - Gabriell (gabriellpt04)
 Portuguese, Brazilian:
  - Shaolin (admshao)
  - Ramon Mendes (rbrgameplays)
  - Fabio Madia (Shaolin)
  - Burkes
  - Guilherme Dias (ThisGuy)
  - Emanoel Lopes (emanoelopes)
  - REMOVED_USER
  - TFSThiagoBR98
  - CaioWzy
  - Vainock
  - mizifih
  - André Gama (ToeOficial)
  - Avellar (BetaTester)
  - clr0dr1g
  - aalonsomb
  - Gol D. Ace (goldace)
  - REMOVED_USER
  - DaDodger
  - DevilLorde
  - João (florretardada)
  - Ramon Gonzalez (ramon200000)
  - Esdras Tarsis (esdrastarsis)
  - dodgepong
  - sauloleite_quim
 Punjabi:
  - manjotsingh0202
 Romanian:
  - Cristian Silaghi (stelistcristi)
  - banrek
  - Hisashi
  - Andrei Ionescu (abcdedjdmddx)
  - BlakeNowah
  - Victor Paul (corvinpaul)
  - Skellytone
  - Mihai G (babasghenciu)
  - Vainock
  - Gol D. Ace (goldace)
  - REMOVED_USER
  - Diamyx
  - dodgepong
  - chaironeko
  - Andrei Daniel (AndrelinYT)
 Russian:
  - Alek Nirov (dectanova)
  - iltrof
  - Andrei Stepanov (adem4ik)
  - Pavel (Shevalie)
  - VNGXR
  - Bugo (Bugo)
  - MaximGribanov
  - dodgepong
  - Vainock
  - Gol D. Ace (goldace)
  - Әлмәт Ак Арыслан (19082004amir)
  - Runoff Screen (glebpozbnakov62)
  - Yaroslav (MrYadro)
  - fromgate (fromgate)
  - Strange Grey Cat (StrangeGreyCat)
  - Artem4ik
  - Sirboys
  - REMOVED_USER
  - Andy (anry025)
  - Nemesis (Nemesis07)
  - Serge Sklyarov (sergesklyarov)
  - Makatavin # (makatavin0)
  - ExZo (ExZo)
  - Vlad (KoTmaxHo)
  - Vladimir (jeffors)
  - Myasko
  - Mixaill
  - Nikita Bibanaev (nicky18013)
  - Sergei Fug1t1v3 (fug)
  - Walt Gee (vovanych)
  - allan walpy (AndreyLysenkov)
  - Drahonn
  - SandoBY
  - Yuri Mihaqlov (yurijmi)
  - TR1D
  - Sigge Stjärnholm (Kladdy)
  - iLefty
  - Evgeny Bogdanov (vtrifonov548)
  - Kuji Kitamura (KujiKita)
  - MUHADDIS MEDIA (muhaddismedia)
  - Milise_Tailise
  - Churikov Radmir Sergeevich (RadMir2)
  - chaironeko
  - Юрій (Devinit)
  - L1Q
  - Sasha Sorokin (DaFri-Nochiterov)
 Scottish Gaelic:
  - GunChleoc
  - Vainock
  - REMOVED_USER
  - Gol D. Ace (goldace)
 Serbian (Cyrillic):
  - nikolanikola
  - medicmomcilo
  - Vainock
  - Acamicamacaraca
  - Gol D. Ace (goldace)
  - veles330
  - LittleGirl_WithPonyTail (alexs1320)
  - Лука Живановић (lukazivanovic)
  - REMOVED_USER
  - dodgepong
  - scienceangel
 Serbian (Latin):
  - nikolanikola
  - medicmomcilo
  - LittleGirl_WithPonyTail (alexs1320)
  - Vainock
  - Gol D. Ace (goldace)
  - Rale Sarcevic (ralesarcevic)
  - REMOVED_USER
  - dodgepong
  - scienceangel
 Slovak:
  - Luki (luki1412)
  - Erik Bročko (ericek111)
  - Ján M (longmoped)
  - Anton Lokaj (anlo)
  - Vainock
  - pelledani_
  - Gol D. Ace (goldace)
  - LoLLy Nka (lollynka279)
  - REMOVED_USER
  - dodgepong
 Slovenian:
  - REMOVED_USER
  - Arnold Marko (atomicmind)
  - kristjan.krusic (krusic22)
  - MG lolenstine (mglolenstine)
  - REMOVED_USER
  - Vainock
  - Grimpy
  - REMOVED_USER
  - Gol D. Ace (goldace)
  - ArcaneWater
  - REMOVED_USER
  - dodgepong
 Southern Sotho:
  - Samuel Nthoroane (Samuel_Nthoroane)
  - Gol D. Ace (goldace)
 Spanish:
  - Marcos Vidal Martinez (M4RK22)
  - Jaime Martinez Rincon (mrjaime1999)
  - Monsteer
  - Roberto Lorenzo (HonzoNebro)
  - Jaime Muñoz Martín (jmmartin_5)
  - Dalia Sofía Magallón Páramo (SweetSofiMC)
  - Alex E. D. B. (alexedb)
  - EG Gamer (eggamer131)
  - REMOVED_USER
  - Pilar G. (TheMadnessLady)
  - Gol D. Ace (goldace)
  - Vainock
  - Maximiliano Schtroumpftech Pena-Roig (som2tokmynam)
  - Carlos Plata (carlosesgenial33)
  - REMOVED_USER
  - eemiroj
  - henrycontreras
  - Skiddome (sergiomalagonmartin)
  - Marcos Vidal (markitos.maki22)
  - Ruben Deig Ramos (rdeigramos)
  - Ray (Ipsumry)
  - makiza1 (micosil_2)
  - Santiago Pereyra (SannttVIII)
  - jan (test83318)
  - David Sonico (davidsubsonico)
  - Eleazar (MtrElee3)
  - amssusgameplays (willifake052)
  - Jaire (corpi.98)
  - Sigge Stjärnholm (Kladdy)
  - dodgepong
  - Rodrigo Ipince (ipince)
  - chaironeko
  - Rubén Pérez (RixzZ)
  - Rumope
  - SneakyFish5
  - Stuttero
 Swedish:
  - Anton R (FirePhoenix)
  - Sigge Stjärnholm (Kladdy)
  - Bjorn Astrom (beeanyew)
  - Laccy IEST (Laccy)
  - 0x9fff00
  - Gustav Ekner (ekner)
  - Vainock
  - Jonatan Nyberg (sweuser)
  - ArvidTheSwe
  - REMOVED_USER
  - Olle Dahström (odahlstrom)
  - Gol D. Ace (goldace)
  - Henrik Mattsson-Mårn (rchk)
  - chaironeko
  - Jonas Svensson (jonassanojj99)
  - Hannes Blåman (thebluis)
  - TacticalKebab
  - dodgepong
  - granbom
  - MaltahlGaming (maltahlgaming)
  - Carl Fürstenberg (azatoth)
 Tagalog:
  - dandalion
  - jermel
  - jbeguna04
  - philiparniebinag
  - Vainock
  - Gol D. Ace (goldace)
  - Red Dayao (steemitph)
  - REMOVED_USER
  - Laarnice
  - Raylir
 Tamil:
  - anto27
  - REMOVED_USER
  - REMOVED_USER
  - Gol D. Ace (goldace)
  - Vainock
 Thai:
  - Nawin Somprasong (thaipirch98)
  - ธีรภัทร์ โยชนา (Gataro)
  - sakuhanachan* (sakuhanachanloli)
  - Sakia Normal Human (arcanaarcana5)
  - 盛凤阁 (execzero)
  - nongnoobjung (kitcharuk_4)
  - แมน ทศพร (lovemanna456)
  - REMOVED_USER
  - Vainock
  - dodgepong
  - Gol D. Ace (goldace)
 Turkish:
  - monolifed
  - Ali Kömesöğütlü (Mobile46) (byzlo685)
  - omer.karagoz (mrkaragoz)
  - Kizil Elma (tcerit87)
  - REMOVED_USER
  - Cemal Dursun (cmldrs)
  - Savas Tokmak (Laserist)
  - Umut kılıç (kilic190787)
  - furkanbicici (furkanbicici)
  - Murat Karagöz (anemon_1994)
  - Vainock
  - REMOVED_USER
  - Alperen Yıldız (Sparrow34)
  - berkcan uçan (ibnehayati)
  - Gol D. Ace (goldace)
  - gecebekcisi1
  - mustafaa
  - Richard Stanway (r1ch)
  - REMOVED_USER
  - Khedi
  - Hydroboost (Hydroboost)
  - Mustafa Arslan (mstfaa)
  - Alican Gultekin (Vitaefinis)
  - REMOVED_USER
  - Türker Yıldırım (turkeryildirim)
  - basakbk
  - chaironeko
  - dodgepong
 Ukrainian:
  - SuslikV
  - Lino Bico (bicolino34)
  - Vainock
  - REMOVED_USER
  - REMOVED_USER
  - Юрій (Devinit)
  - Gol D. Ace (goldace)
  - Andy (anry025)
  - NoPressure
  - បងមាន តែអូន (cheaiphone267)
  - geimfis
  - L1Q
  - powerdef
  - Maksym Tymoshyk (maximillian_)
  - dodgepong
 Urdu (Pakistan):
  - Rana Awais (ehtisham)
  - tahirsada
  - shamuntohamd
  - Gol D. Ace (goldace)
 Vietnamese:
  - Johnny “max20091” Utah (boostyourprogram)
  - Hưng Nguyễn (hoyostudio)
  - IoeCmcomc (ioecmcomc)
  - ngoisaosang (ngoisaosang)
  - REMOVED_USER
  - Gol D. Ace (goldace)
  - Drake Strike (phjtieudoc)
  - BIGO - 지혜 (parkjihye96)
  - Blog Đào Lê Minh (daoleminh2010)
  - REMOVED_USER
  - Hà Phi Hùng (haphihungcom)
  - Vainock
  - Vũ Hải Tây (tayngungo1999)
  - Dawkin Nguyen (dawkinit)
  - dodgepong
  - NCAA (NCAA)

hr() {
  echo "───────────────────────────────────────────────────"
  echo $1
  echo "───────────────────────────────────────────────────"
}

# Exit if something fails
set -e

# Generate file name variables
export GIT_TAG=$(git describe --abbrev=0)
export GIT_HASH=$(git rev-parse --short HEAD)
export FILE_DATE=$(date +%Y-%m-%d.%H-%M-%S)
export FILENAME=$FILE_DATE-$GIT_HASH-$TRAVIS_BRANCH-osx.dmg

echo "git tag: $GIT_TAG"

cd ./build

# Move obslua
hr "Moving OBS LUA"
mv ./rundir/RelWithDebInfo/data/obs-scripting/obslua.so ./rundir/RelWithDebInfo/bin/

# Move obspython
hr "Moving OBS Python"
mv ./rundir/RelWithDebInfo/data/obs-scripting/_obspython.so ./rundir/RelWithDebInfo/bin/
mv ./rundir/RelWithDebInfo/data/obs-scripting/obspython.py ./rundir/RelWithDebInfo/bin/

# Package everything into a nice .app
hr "Packaging .app"
STABLE=false
if [ -n "${TRAVIS_TAG}" ]; then
  STABLE=true
fi

#sudo python ../CI/install/osx/build_app.py --public-key ../CI/install/osx/OBSPublicDSAKey.pem --sparkle-framework ../../sparkle/Sparkle.framework --stable=$STABLE

../CI/install/osx/packageApp.sh

# fix obs outputs plugin it doesn't play nicely with dylibBundler at the moment
cp /usr/local/opt/mbedtls/lib/libmbedtls.12.dylib ./OBS.app/Contents/Frameworks/
cp /usr/local/opt/mbedtls/lib/libmbedcrypto.3.dylib ./OBS.app/Contents/Frameworks/
cp /usr/local/opt/mbedtls/lib/libmbedx509.0.dylib ./OBS.app/Contents/Frameworks/
chmod +w ./OBS.app/Contents/Frameworks/*.dylib
install_name_tool -id @executable_path/../Frameworks/libmbedtls.12.dylib ./OBS.app/Contents/Frameworks/libmbedtls.12.dylib
install_name_tool -id @executable_path/../Frameworks/libmbedcrypto.3.dylib ./OBS.app/Contents/Frameworks/libmbedcrypto.3.dylib
install_name_tool -id @executable_path/../Frameworks/libmbedx509.0.dylib ./OBS.app/Contents/Frameworks/libmbedx509.0.dylib
install_name_tool -change libmbedtls.12.dylib @executable_path/../Frameworks/libmbedtls.12.dylib ./OBS.app/Contents/Plugins/obs-outputs.so
install_name_tool -change libmbedcrypto.3.dylib @executable_path/../Frameworks/libmbedcrypto.3.dylib ./OBS.app/Contents/Plugins/obs-outputs.so
install_name_tool -change libmbedx509.0.dylib @executable_path/../Frameworks/libmbedx509.0.dylib ./OBS.app/Contents/Plugins/obs-outputs.so
install_name_tool -change /usr/local/opt/curl/lib/libcurl.4.dylib @executable_path/../Frameworks/libcurl.4.dylib ./OBS.app/Contents/Plugins/obs-outputs.so
install_name_tool -change @rpath/libobs.0.dylib @executable_path/../Frameworks/libobs.0.dylib ./OBS.app/Contents/Plugins/obs-outputs.so
install_name_tool -change /tmp/obsdeps/bin/libjansson.4.dylib @executable_path/../Frameworks/libjansson.4.dylib ./OBS.app/Contents/Plugins/obs-outputs.so

# copy sparkle into the app
hr "Copying Sparkle.framework"
cp -R ../../sparkle/Sparkle.framework ./OBS.app/Contents/Frameworks/
install_name_tool -change @rpath/Sparkle.framework/Versions/A/Sparkle @executable_path/../Frameworks/Sparkle.framework/Versions/A/Sparkle ./OBS.app/Contents/MacOS/obs

# Copy Chromium embedded framework to app Frameworks directory
hr "Copying Chromium Embedded Framework.framework"
sudo mkdir -p OBS.app/Contents/Frameworks
sudo cp -R ../../cef_binary_${CEF_BUILD_VERSION}_macosx64/Release/Chromium\ Embedded\ Framework.framework OBS.app/Contents/Frameworks/

install_name_tool -change /usr/local/opt/qt/lib/QtGui.framework/Versions/5/QtGui @executable_path/../Frameworks/QtGui.framework/Versions/5/QtGui ./OBS.app/Contents/Plugins/obs-browser.so
install_name_tool -change /usr/local/opt/qt/lib/QtCore.framework/Versions/5/QtCore @executable_path/../Frameworks/QtCore.framework/Versions/5/QtCore ./OBS.app/Contents/Plugins/obs-browser.so
install_name_tool -change /usr/local/opt/qt/lib/QtWidgets.framework/Versions/5/QtWidgets @executable_path/../Frameworks/QtWidgets.framework/Versions/5/QtWidgets ./OBS.app/Contents/Plugins/obs-browser.so

cp ../CI/install/osx/OBSPublicDSAKey.pem OBS.app/Contents/Resources

# edit plist
plutil -insert CFBundleVersion -string $GIT_TAG ./OBS.app/Contents/Info.plist
plutil -insert CFBundleShortVersionString -string $GIT_TAG ./OBS.app/Contents/Info.plist
plutil -insert OBSFeedsURL -string https://obsproject.com/osx_update/feeds.xml ./OBS.app/Contents/Info.plist
plutil -insert SUFeedURL -string https://obsproject.com/osx_update/stable/updates.xml ./OBS.app/Contents/Info.plist
plutil -insert SUPublicDSAKeyFile -string OBSPublicDSAKey.pem ./OBS.app/Contents/Info.plist

dmgbuild -s ../CI/install/osx/settings.json "OBS" obs.dmg

if [ -v "$TRAVIS" ]; then
	# Signing stuff
	hr "Decrypting Cert"
	openssl aes-256-cbc -K $encrypted_dd3c7f5e9db9_key -iv $encrypted_dd3c7f5e9db9_iv -in ../CI/osxcert/Certificates.p12.enc -out Certificates.p12 -d
	hr "Creating Keychain"
	security create-keychain -p mysecretpassword build.keychain
	security default-keychain -s build.keychain
	security unlock-keychain -p mysecretpassword build.keychain
	security set-keychain-settings -t 3600 -u build.keychain
	hr "Importing certs into keychain"
	security import ./Certificates.p12 -k build.keychain -T /usr/bin/productsign -P ""
	# macOS 10.12+
	security set-key-partition-list -S apple-tool:,apple: -s -k mysecretpassword build.keychain
fi

cp ./OBS.dmg ./$FILENAME

# Move to the folder that travis uses to upload artifacts from
hr "Moving package to nightly folder for distribution"
mkdir ../nightly
sudo mv ./$FILENAME ../nightly


robocopy .\build32\rundir\RelWithDebInfo .\build\ /E /XF .gitignore
robocopy .\build64\rundir\RelWithDebInfo .\build\ /E /XC /XN /XO /XF .gitignore
7z a build.zip .\build\*

#!/bin/bash

set -ex
ccache -s || echo "CCache is not available."
mkdir build && cd build
cmake -DBUILD_CAPTIONS=ON -DBUILD_BROWSER=ON -DCEF_ROOT_DIR="../cef_binary_${CEF_BUILD_VERSION}_linux64" ..


# Make sure ccache is found
export PATH=/usr/local/opt/ccache/libexec:$PATH

git fetch --tags

mkdir build
cd build
cmake -DENABLE_SPARKLE_UPDATER=ON \
-DCMAKE_OSX_DEPLOYMENT_TARGET=10.11 \
-DQTDIR=/usr/local/Cellar/qt/5.14.1 \
-DDepsPath=/tmp/obsdeps \
-DVLCPath=$PWD/../../vlc-3.0.8 \
-DBUILD_BROWSER=ON \
-DBROWSER_DEPLOY=ON \
-DBUILD_CAPTIONS=ON \
-DWITH_RTMPS=ON \
-DCEF_ROOT_DIR=$PWD/../../cef_binary_${CEF_BUILD_VERSION}_macosx64 ..

#!/bin/bash
dirty=$(git ls-files --modified)

set +x
if [[ $dirty ]]; then
	echo "================================="
    echo "Files were not formatted properly"
    echo "$dirty"
    echo "================================="
    exit 1
fi

#!/bin/sh
set -ex

curl -L https://packagecloud.io/github/git-lfs/gpgkey | sudo apt-key add -

sudo apt-get -qq update
sudo apt-get install -y \
        build-essential \
        checkinstall \
        cmake \
        libasound2-dev \
        libavcodec-dev \
        libavdevice-dev \
        libavfilter-dev \
        libavformat-dev \
        libavutil-dev \
        libcurl4-openssl-dev \
        libfdk-aac-dev \
        libfontconfig-dev \
        libfreetype6-dev \
        libgl1-mesa-dev \
        libjack-jackd2-dev \
        libjansson-dev \
        libluajit-5.1-dev \
        libpulse-dev \
        libqt5x11extras5-dev \
        libspeexdsp-dev \
        libswresample-dev \
        libswscale-dev \
        libudev-dev \
        libv4l-dev \
        libva-dev \
        libvlc-dev \
        libx11-dev \
        libx264-dev \
        libxcb-randr0-dev \
        libxcb-shm0-dev \
        libxcb-xinerama0-dev \
        libxcomposite-dev \
        libxinerama-dev \
        libmbedtls-dev \
        pkg-config \
        python3-dev \
        qtbase5-dev \
        libqt5svg5-dev \
        swig

# build cef
wget --quiet --retry-connrefused --waitretry=1 https://cdn-fastly.obsproject.com/downloads/cef_binary_${CEF_BUILD_VERSION}_linux64.tar.bz2
tar -xjf ./cef_binary_${CEF_BUILD_VERSION}_linux64.tar.bz2

hr() {
  echo "───────────────────────────────────────────────────"
  echo $1
  echo "───────────────────────────────────────────────────"
}

# Exit if something fails
set -e

# Echo all commands before executing
set -v

if [[ $TRAVIS ]]; then
  git fetch --unshallow
fi

git fetch origin --tags

# Leave obs-studio folder
cd ../

# Install Packages app so we can build a package later
# http://s.sudre.free.fr/Software/Packages/about.html
hr "Downloading Packages app"
wget --quiet --retry-connrefused --waitretry=1 https://s3-us-west-2.amazonaws.com/obs-nightly/Packages.pkg
sudo installer -pkg ./Packages.pkg -target /

brew update

#Base OBS Deps and ccache
brew install jack speexdsp ccache mbedtls freetype fdk-aac
brew install https://gist.githubusercontent.com/DDRBoxman/9c7a2b08933166f4b61ed9a44b242609/raw/ef4de6c587c6bd7f50210eccd5bd51ff08e6de13/qt.rb
if [ -d "$(brew --cellar)/swig" ]; then
    brew unlink swig
fi
brew install https://gist.githubusercontent.com/DDRBoxman/4cada55c51803a2f963fa40ce55c9d3e/raw/572c67e908bfbc1bcb8c476ea77ea3935133f5b5/swig.rb

pip install dmgbuild

export PATH=/usr/local/opt/ccache/libexec:$PATH
ccache -s || echo "CCache is not available."

# Fetch and untar prebuilt OBS deps that are compatible with older versions of OSX
hr "Downloading OBS deps"
wget --quiet --retry-connrefused --waitretry=1 https://github.com/obsproject/obs-deps/releases/download/2020-04-24/osx-deps-2020-04-24.tar.gz
tar -xf ./osx-deps-2020-04-24.tar.gz -C /tmp

# Fetch vlc codebase
hr "Downloading VLC repo"
wget --quiet --retry-connrefused --waitretry=1 https://downloads.videolan.org/vlc/3.0.8/vlc-3.0.8.tar.xz
tar -xf vlc-3.0.8.tar.xz

# Get sparkle
hr "Downloading Sparkle framework"
wget --quiet --retry-connrefused --waitretry=1 -O sparkle.tar.bz2 https://github.com/sparkle-project/Sparkle/releases/download/1.23.0/Sparkle-1.23.0.tar.bz2
mkdir ./sparkle
tar -xf ./sparkle.tar.bz2 -C ./sparkle
sudo cp -R ./sparkle/Sparkle.framework /Library/Frameworks/Sparkle.framework

# CEF Stuff
hr "Downloading CEF"
wget --quiet --retry-connrefused --waitretry=1 https://obs-nightly.s3-us-west-2.amazonaws.com/cef_binary_${CEF_BUILD_VERSION}_macosx64.tar.bz2
tar -xf ./cef_binary_${CEF_BUILD_VERSION}_macosx64.tar.bz2
cd ./cef_binary_${CEF_BUILD_VERSION}_macosx64
# remove a broken test
sed -i '.orig' '/add_subdirectory(tests\/ceftests)/d' ./CMakeLists.txt
# target 10.11
sed -i '.orig' s/\"10.9\"/\"10.11\"/ ./cmake/cef_variables.cmake
mkdir build
cd ./build
cmake -DCMAKE_CXX_FLAGS="-std=c++11 -stdlib=libc++" -DCMAKE_EXE_LINKER_FLAGS="-std=c++11 -stdlib=libc++" -DCMAKE_OSX_DEPLOYMENT_TARGET=10.11 ..
make -j4
mkdir libcef_dll
cd ../../

if exist Qt_5.10.1.7z (curl -kLO https://cdn-fastly.obsproject.com/downloads/Qt_5.10.1.7z -f --retry 5 -z Qt_5.10.1.7z) else (curl -kLO https://cdn-fastly.obsproject.com/downloads/Qt_5.10.1.7z -f --retry 5 -C -)
7z x Qt_5.10.1.7z -oQt
mv Qt C:\QtDep
dir C:\QtDep

#!/bin/sh
set -ex

build_config=RelWithDebInfo

if exist dependencies2017.zip (curl -kLO https://cdn-fastly.obsproject.com/downloads/dependencies2017.zip -f --retry 5 -z dependencies2017.zip) else (curl -kLO https://cdn-fastly.obsproject.com/downloads/dependencies2017.zip -f --retry 5 -C -)
if exist vlc.zip (curl -kLO https://cdn-fastly.obsproject.com/downloads/vlc.zip -f --retry 5 -z vlc.zip) else (curl -kLO https://cdn-fastly.obsproject.com/downloads/vlc.zip -f --retry 5 -C -)
if exist cef_binary_%CEF_VERSION%_windows32_minimal.zip (curl -kLO https://cdn-fastly.obsproject.com/downloads/cef_binary_%CEF_VERSION%_windows32_minimal.zip -f --retry 5 -z cef_binary_%CEF_VERSION%_windows32_minimal.zip) else (curl -kLO https://cdn-fastly.obsproject.com/downloads/cef_binary_%CEF_VERSION%_windows32_minimal.zip -f --retry 5 -C -)
if exist cef_binary_%CEF_VERSION%_windows64_minimal.zip (curl -kLO https://cdn-fastly.obsproject.com/downloads/cef_binary_%CEF_VERSION%_windows64_minimal.zip -f --retry 5 -z cef_binary_%CEF_VERSION%_windows64_minimal.zip) else (curl -kLO https://cdn-fastly.obsproject.com/downloads/cef_binary_%CEF_VERSION%_windows64_minimal.zip -f --retry 5 -C -)
7z x dependencies2017.zip -odependencies2017
7z x vlc.zip -ovlc
7z x cef_binary_%CEF_VERSION%_windows32_minimal.zip -oCEF_32
7z x cef_binary_%CEF_VERSION%_windows64_minimal.zip -oCEF_64
set DepsPath32=%CD%\dependencies2017\win32
set DepsPath64=%CD%\dependencies2017\win64
set VLCPath=%CD%\vlc
set QTDIR32=C:\QtDep\5.10.1\msvc2017
set QTDIR64=C:\QtDep\5.10.1\msvc2017_64
set CEF_32=%CD%\CEF_32\cef_binary_%CEF_VERSION%_windows32_minimal
set CEF_64=%CD%\CEF_64\cef_binary_%CEF_VERSION%_windows64_minimal
set build_config=RelWithDebInfo
mkdir build build32 build64
if "%TWITCH-CLIENTID%"=="$(twitch_clientid)" (
cd ./build32
cmake -G "Visual Studio 16 2019" -A Win32 -DCMAKE_SYSTEM_VERSION=10.0 -DCOPIED_DEPENDENCIES=false -DCOPY_DEPENDENCIES=true -DBUILD_CAPTIONS=true -DCOMPILE_D3D12_HOOK=true -DBUILD_BROWSER=true -DCEF_ROOT_DIR=%CEF_32% ..
cd ../build64
cmake -G "Visual Studio 16 2019" -A x64 -DCMAKE_SYSTEM_VERSION=10.0 -DCOPIED_DEPENDENCIES=false -DCOPY_DEPENDENCIES=true -DBUILD_CAPTIONS=true -DCOMPILE_D3D12_HOOK=true -DBUILD_BROWSER=true -DCEF_ROOT_DIR=%CEF_64% ..
) else (
cd ./build32
cmake -G "Visual Studio 16 2019" -A Win32 -DCMAKE_SYSTEM_VERSION=10.0 -DCOPIED_DEPENDENCIES=false -DCOPY_DEPENDENCIES=true -DBUILD_CAPTIONS=true -DCOMPILE_D3D12_HOOK=true -DBUILD_BROWSER=true -DCEF_ROOT_DIR=%CEF_32% -DTWITCH_CLIENTID="%TWITCH-CLIENTID%" -DTWITCH_HASH="%TWITCH-HASH%" -DMIXER_CLIENTID="%MIXER-CLIENTID%" -DMIXER_HASH="%MIXER-HASH%" -DRESTREAM_CLIENTID="%RESTREAM-CLIENTID%" -DRESTREAM_HASH="%RESTREAM-HASH%" ..
cd ../build64
cmake -G "Visual Studio 16 2019" -A x64 -DCMAKE_SYSTEM_VERSION=10.0 -DCOPIED_DEPENDENCIES=false -DCOPY_DEPENDENCIES=true -DBUILD_CAPTIONS=true -DCOMPILE_D3D12_HOOK=true -DBUILD_BROWSER=true -DCEF_ROOT_DIR=%CEF_64% -DTWITCH_CLIENTID="%TWITCH-CLIENTID%" -DTWITCH_HASH="%TWITCH-HASH%" -DMIXER_CLIENTID="%MIXER-CLIENTID%" -DMIXER_HASH="%MIXER-HASH%" -DRESTREAM_CLIENTID="%RESTREAM-CLIENTID%" -DRESTREAM_HASH="%RESTREAM-HASH%" ..
)
cd ..

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>PACKAGES</key>
	<array>
		<dict>
			<key>PACKAGE_FILES</key>
			<dict>
				<key>DEFAULT_INSTALL_LOCATION</key>
				<string>/</string>
				<key>HIERARCHY</key>
				<dict>
					<key>CHILDREN</key>
					<array>
						<dict>
							<key>CHILDREN</key>
							<array>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>80</integer>
									<key>PATH</key>
									<string>../../../build/OBS.app</string>
									<key>PATH_TYPE</key>
									<integer>3</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>3</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>80</integer>
									<key>PATH</key>
									<string>Utilities</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
							</array>
							<key>GID</key>
							<integer>80</integer>
							<key>PATH</key>
							<string>Applications</string>
							<key>PATH_TYPE</key>
							<integer>0</integer>
							<key>PERMISSIONS</key>
							<integer>509</integer>
							<key>TYPE</key>
							<integer>1</integer>
							<key>UID</key>
							<integer>0</integer>
						</dict>
						<dict>
							<key>CHILDREN</key>
							<array>
								<dict>
									<key>CHILDREN</key>
									<array>
										<dict>
											<key>CHILDREN</key>
											<array>
												<dict>
													<key>CHILDREN</key>
													<array>
														<dict>
															<key>CHILDREN</key>
															<array>
																<dict>
																	<key>CHILDREN</key>
																	<array>
																		<dict>
																			<key>CHILDREN</key>
																			<array/>
																			<key>GID</key>
																			<integer>80</integer>
																			<key>PATH</key>
																			<string>../../../build/plugins/obs-browser/obs-browser-page</string>
																			<key>PATH_TYPE</key>
																			<integer>3</integer>
																			<key>PERMISSIONS</key>
																			<integer>493</integer>
																			<key>TYPE</key>
																			<integer>3</integer>
																			<key>UID</key>
																			<integer>0</integer>
																		</dict>
																		<dict>
																			<key>CHILDREN</key>
																			<array/>
																			<key>GID</key>
																			<integer>80</integer>
																			<key>PATH</key>
																			<string>../../../build/plugins/obs-browser/obs-browser.so</string>
																			<key>PATH_TYPE</key>
																			<integer>3</integer>
																			<key>PERMISSIONS</key>
																			<integer>493</integer>
																			<key>TYPE</key>
																			<integer>3</integer>
																			<key>UID</key>
																			<integer>0</integer>
																		</dict>
																	</array>
																	<key>GID</key>
																	<integer>80</integer>
																	<key>PATH</key>
																	<string>bin</string>
																	<key>PATH_TYPE</key>
																	<integer>0</integer>
																	<key>PERMISSIONS</key>
																	<integer>493</integer>
																	<key>TYPE</key>
																	<integer>2</integer>
																	<key>UID</key>
																	<integer>0</integer>
																</dict>
															</array>
															<key>GID</key>
															<integer>80</integer>
															<key>PATH</key>
															<string>obs-browser</string>
															<key>PATH_TYPE</key>
															<integer>0</integer>
															<key>PERMISSIONS</key>
															<integer>493</integer>
															<key>TYPE</key>
															<integer>2</integer>
															<key>UID</key>
															<integer>0</integer>
														</dict>
													</array>
													<key>GID</key>
													<integer>80</integer>
													<key>PATH</key>
													<string>plugins</string>
													<key>PATH_TYPE</key>
													<integer>0</integer>
													<key>PERMISSIONS</key>
													<integer>493</integer>
													<key>TYPE</key>
													<integer>2</integer>
													<key>UID</key>
													<integer>0</integer>
												</dict>
											</array>
											<key>GID</key>
											<integer>80</integer>
											<key>PATH</key>
											<string>obs-studio</string>
											<key>PATH_TYPE</key>
											<integer>0</integer>
											<key>PERMISSIONS</key>
											<integer>493</integer>
											<key>TYPE</key>
											<integer>2</integer>
											<key>UID</key>
											<integer>0</integer>
										</dict>
									</array>
									<key>GID</key>
									<integer>80</integer>
									<key>PATH</key>
									<string>Application Support</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>Documentation</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>Filesystems</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>Frameworks</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>Input Methods</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>Internet Plug-Ins</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>LaunchAgents</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>LaunchDaemons</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>PreferencePanes</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>Preferences</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>80</integer>
									<key>PATH</key>
									<string>Printers</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>PrivilegedHelperTools</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>QuickLook</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>QuickTime</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>Screen Savers</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>Scripts</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>Services</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>Widgets</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
							</array>
							<key>GID</key>
							<integer>0</integer>
							<key>PATH</key>
							<string>Library</string>
							<key>PATH_TYPE</key>
							<integer>0</integer>
							<key>PERMISSIONS</key>
							<integer>493</integer>
							<key>TYPE</key>
							<integer>1</integer>
							<key>UID</key>
							<integer>0</integer>
						</dict>
						<dict>
							<key>CHILDREN</key>
							<array>
								<dict>
									<key>CHILDREN</key>
									<array>
										<dict>
											<key>CHILDREN</key>
											<array/>
											<key>GID</key>
											<integer>0</integer>
											<key>PATH</key>
											<string>Extensions</string>
											<key>PATH_TYPE</key>
											<integer>0</integer>
											<key>PERMISSIONS</key>
											<integer>493</integer>
											<key>TYPE</key>
											<integer>1</integer>
											<key>UID</key>
											<integer>0</integer>
										</dict>
									</array>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>Library</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>493</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
							</array>
							<key>GID</key>
							<integer>0</integer>
							<key>PATH</key>
							<string>System</string>
							<key>PATH_TYPE</key>
							<integer>0</integer>
							<key>PERMISSIONS</key>
							<integer>493</integer>
							<key>TYPE</key>
							<integer>1</integer>
							<key>UID</key>
							<integer>0</integer>
						</dict>
						<dict>
							<key>CHILDREN</key>
							<array>
								<dict>
									<key>CHILDREN</key>
									<array/>
									<key>GID</key>
									<integer>0</integer>
									<key>PATH</key>
									<string>Shared</string>
									<key>PATH_TYPE</key>
									<integer>0</integer>
									<key>PERMISSIONS</key>
									<integer>1023</integer>
									<key>TYPE</key>
									<integer>1</integer>
									<key>UID</key>
									<integer>0</integer>
								</dict>
							</array>
							<key>GID</key>
							<integer>80</integer>
							<key>PATH</key>
							<string>Users</string>
							<key>PATH_TYPE</key>
							<integer>0</integer>
							<key>PERMISSIONS</key>
							<integer>493</integer>
							<key>TYPE</key>
							<integer>1</integer>
							<key>UID</key>
							<integer>0</integer>
						</dict>
					</array>
					<key>GID</key>
					<integer>0</integer>
					<key>PATH</key>
					<string>/</string>
					<key>PATH_TYPE</key>
					<integer>0</integer>
					<key>PERMISSIONS</key>
					<integer>493</integer>
					<key>TYPE</key>
					<integer>1</integer>
					<key>UID</key>
					<integer>0</integer>
				</dict>
				<key>PAYLOAD_TYPE</key>
				<integer>0</integer>
				<key>VERSION</key>
				<integer>2</integer>
			</dict>
			<key>PACKAGE_SCRIPTS</key>
			<dict>
				<key>POSTINSTALL_PATH</key>
				<dict>
					<key>PATH</key>
					<string>post-install.sh</string>
					<key>PATH_TYPE</key>
					<integer>3</integer>
				</dict>
				<key>PREINSTALL_PATH</key>
				<dict/>
				<key>RESOURCES</key>
				<array/>
			</dict>
			<key>PACKAGE_SETTINGS</key>
			<dict>
				<key>AUTHENTICATION</key>
				<integer>1</integer>
				<key>CONCLUSION_ACTION</key>
				<integer>0</integer>
				<key>IDENTIFIER</key>
				<string>org.obsproject.pkg.obs-studio</string>
				<key>NAME</key>
				<string>OBS</string>
				<key>OVERWRITE_PERMISSIONS</key>
				<false/>
				<key>VERSION</key>
				<string>1.0</string>
			</dict>
			<key>UUID</key>
			<string>19CCE3F2-8911-4364-B673-8B5BC3ABD4DA</string>
		</dict>
		<dict>
			<key>PACKAGE_SETTINGS</key>
			<dict>
				<key>LOCATION</key>
				<integer>0</integer>
				<key>NAME</key>
				<string>SyphonInject</string>
			</dict>
			<key>PATH</key>
			<dict>
				<key>PATH</key>
				<string>SyphonInject.pkg</string>
				<key>PATH_TYPE</key>
				<integer>1</integer>
			</dict>
			<key>TYPE</key>
			<integer>1</integer>
			<key>UUID</key>
			<string>0CC9C67E-4D14-4794-9930-019925513B1C</string>
		</dict>
	</array>
	<key>PROJECT</key>
	<dict>
		<key>PROJECT_COMMENTS</key>
		<dict>
			<key>NOTES</key>
			<data>
			PCFET0NUWVBFIGh0bWwgUFVCTElDICItLy9XM0MvL0RURCBIVE1M
			IDQuMDEvL0VOIiAiaHR0cDovL3d3dy53My5vcmcvVFIvaHRtbDQv
			c3RyaWN0LmR0ZCI+CjxodG1sPgo8aGVhZD4KPG1ldGEgaHR0cC1l
			cXVpdj0iQ29udGVudC1UeXBlIiBjb250ZW50PSJ0ZXh0L2h0bWw7
			IGNoYXJzZXQ9VVRGLTgiPgo8bWV0YSBodHRwLWVxdWl2PSJDb250
			ZW50LVN0eWxlLVR5cGUiIGNvbnRlbnQ9InRleHQvY3NzIj4KPHRp
			dGxlPjwvdGl0bGU+CjxtZXRhIG5hbWU9IkdlbmVyYXRvciIgY29u
			dGVudD0iQ29jb2EgSFRNTCBXcml0ZXIiPgo8bWV0YSBuYW1lPSJD
			b2NvYVZlcnNpb24iIGNvbnRlbnQ9IjE1MDQuODEiPgo8c3R5bGUg
			dHlwZT0idGV4dC9jc3MiPgo8L3N0eWxlPgo8L2hlYWQ+Cjxib2R5
			Pgo8L2JvZHk+CjwvaHRtbD4K
			</data>
		</dict>
		<key>PROJECT_PRESENTATION</key>
		<dict>
			<key>BACKGROUND</key>
			<dict>
				<key>ALIGNMENT</key>
				<integer>4</integer>
				<key>BACKGROUND_PATH</key>
				<dict>
					<key>PATH</key>
					<string>obs.png</string>
					<key>PATH_TYPE</key>
					<integer>1</integer>
				</dict>
				<key>CUSTOM</key>
				<integer>1</integer>
				<key>SCALING</key>
				<integer>0</integer>
			</dict>
			<key>INSTALLATION TYPE</key>
			<dict>
				<key>HIERARCHIES</key>
				<dict>
					<key>INSTALLER</key>
					<dict>
						<key>LIST</key>
						<array>
							<dict>
								<key>DESCRIPTION</key>
								<array/>
								<key>OPTIONS</key>
								<dict>
									<key>HIDDEN</key>
									<false/>
									<key>STATE</key>
									<integer>0</integer>
								</dict>
								<key>PACKAGE_UUID</key>
								<string>19CCE3F2-8911-4364-B673-8B5BC3ABD4DA</string>
								<key>REQUIREMENTS</key>
								<array/>
								<key>TITLE</key>
								<array/>
								<key>TOOLTIP</key>
								<array/>
								<key>TYPE</key>
								<integer>0</integer>
								<key>UUID</key>
								<string>7C540711-59F4-479C-9CFD-8C4D6594992E</string>
							</dict>
							<dict>
								<key>DESCRIPTION</key>
								<array/>
								<key>OPTIONS</key>
								<dict>
									<key>HIDDEN</key>
									<false/>
									<key>STATE</key>
									<integer>1</integer>
								</dict>
								<key>PACKAGE_UUID</key>
								<string>0CC9C67E-4D14-4794-9930-019925513B1C</string>
								<key>REQUIREMENTS</key>
								<array/>
								<key>TITLE</key>
								<array/>
								<key>TOOLTIP</key>
								<array/>
								<key>TYPE</key>
								<integer>0</integer>
								<key>UUID</key>
								<string>BBDE08F6-D7EE-47CB-881F-7F208B3A604B</string>
							</dict>
						</array>
						<key>REMOVED</key>
						<dict/>
					</dict>
				</dict>
				<key>INSTALLATION TYPE</key>
				<integer>0</integer>
				<key>MODE</key>
				<integer>0</integer>
			</dict>
			<key>INSTALLATION_STEPS</key>
			<array>
				<dict>
					<key>ICPRESENTATION_CHAPTER_VIEW_CONTROLLER_CLASS</key>
					<string>ICPresentationViewIntroductionController</string>
					<key>INSTALLER_PLUGIN</key>
					<string>Introduction</string>
					<key>LIST_TITLE_KEY</key>
					<string>InstallerSectionTitle</string>
				</dict>
				<dict>
					<key>ICPRESENTATION_CHAPTER_VIEW_CONTROLLER_CLASS</key>
					<string>ICPresentationViewReadMeController</string>
					<key>INSTALLER_PLUGIN</key>
					<string>ReadMe</string>
					<key>LIST_TITLE_KEY</key>
					<string>InstallerSectionTitle</string>
				</dict>
				<dict>
					<key>ICPRESENTATION_CHAPTER_VIEW_CONTROLLER_CLASS</key>
					<string>ICPresentationViewLicenseController</string>
					<key>INSTALLER_PLUGIN</key>
					<string>License</string>
					<key>LIST_TITLE_KEY</key>
					<string>InstallerSectionTitle</string>
				</dict>
				<dict>
					<key>ICPRESENTATION_CHAPTER_VIEW_CONTROLLER_CLASS</key>
					<string>ICPresentationViewDestinationSelectController</string>
					<key>INSTALLER_PLUGIN</key>
					<string>TargetSelect</string>
					<key>LIST_TITLE_KEY</key>
					<string>InstallerSectionTitle</string>
				</dict>
				<dict>
					<key>ICPRESENTATION_CHAPTER_VIEW_CONTROLLER_CLASS</key>
					<string>ICPresentationViewInstallationTypeController</string>
					<key>INSTALLER_PLUGIN</key>
					<string>PackageSelection</string>
					<key>LIST_TITLE_KEY</key>
					<string>InstallerSectionTitle</string>
				</dict>
				<dict>
					<key>ICPRESENTATION_CHAPTER_VIEW_CONTROLLER_CLASS</key>
					<string>ICPresentationViewInstallationController</string>
					<key>INSTALLER_PLUGIN</key>
					<string>Install</string>
					<key>LIST_TITLE_KEY</key>
					<string>InstallerSectionTitle</string>
				</dict>
				<dict>
					<key>ICPRESENTATION_CHAPTER_VIEW_CONTROLLER_CLASS</key>
					<string>ICPresentationViewSummaryController</string>
					<key>INSTALLER_PLUGIN</key>
					<string>Summary</string>
					<key>LIST_TITLE_KEY</key>
					<string>InstallerSectionTitle</string>
				</dict>
			</array>
			<key>INTRODUCTION</key>
			<dict>
				<key>LOCALIZATIONS</key>
				<array/>
			</dict>
			<key>LICENSE</key>
			<dict>
				<key>KEYWORDS</key>
				<dict/>
				<key>LOCALIZATIONS</key>
				<array/>
				<key>MODE</key>
				<integer>0</integer>
			</dict>
			<key>README</key>
			<dict>
				<key>LOCALIZATIONS</key>
				<array/>
			</dict>
			<key>SUMMARY</key>
			<dict>
				<key>LOCALIZATIONS</key>
				<array/>
			</dict>
			<key>TITLE</key>
			<dict>
				<key>LOCALIZATIONS</key>
				<array>
					<dict>
						<key>LANGUAGE</key>
						<string>English</string>
						<key>VALUE</key>
						<string>OBS</string>
					</dict>
				</array>
			</dict>
		</dict>
		<key>PROJECT_REQUIREMENTS</key>
		<dict>
			<key>LIST</key>
			<array/>
			<key>POSTINSTALL_PATH</key>
			<dict/>
			<key>PREINSTALL_PATH</key>
			<dict/>
			<key>RESOURCES</key>
			<array/>
			<key>ROOT_VOLUME_ONLY</key>
			<false/>
		</dict>
		<key>PROJECT_SETTINGS</key>
		<dict>
			<key>ADVANCED_OPTIONS</key>
			<dict/>
			<key>BUILD_FORMAT</key>
			<integer>0</integer>
			<key>BUILD_PATH</key>
			<dict>
				<key>PATH</key>
				<string>../../../build</string>
				<key>PATH_TYPE</key>
				<integer>3</integer>
			</dict>
			<key>EXCLUDED_FILES</key>
			<array>
				<dict>
					<key>PATTERNS_ARRAY</key>
					<array>
						<dict>
							<key>REGULAR_EXPRESSION</key>
							<false/>
							<key>STRING</key>
							<string>.DS_Store</string>
							<key>TYPE</key>
							<integer>0</integer>
						</dict>
					</array>
					<key>PROTECTED</key>
					<true/>
					<key>PROXY_NAME</key>
					<string>Remove .DS_Store files</string>
					<key>PROXY_TOOLTIP</key>
					<string>Remove ".DS_Store" files created by the Finder.</string>
					<key>STATE</key>
					<true/>
				</dict>
				<dict>
					<key>PATTERNS_ARRAY</key>
					<array>
						<dict>
							<key>REGULAR_EXPRESSION</key>
							<false/>
							<key>STRING</key>
							<string>.pbdevelopment</string>
							<key>TYPE</key>
							<integer>0</integer>
						</dict>
					</array>
					<key>PROTECTED</key>
					<true/>
					<key>PROXY_NAME</key>
					<string>Remove .pbdevelopment files</string>
					<key>PROXY_TOOLTIP</key>
					<string>Remove ".pbdevelopment" files created by ProjectBuilder or Xcode.</string>
					<key>STATE</key>
					<true/>
				</dict>
				<dict>
					<key>PATTERNS_ARRAY</key>
					<array>
						<dict>
							<key>REGULAR_EXPRESSION</key>
							<false/>
							<key>STRING</key>
							<string>CVS</string>
							<key>TYPE</key>
							<integer>1</integer>
						</dict>
						<dict>
							<key>REGULAR_EXPRESSION</key>
							<false/>
							<key>STRING</key>
							<string>.cvsignore</string>
							<key>TYPE</key>
							<integer>0</integer>
						</dict>
						<dict>
							<key>REGULAR_EXPRESSION</key>
							<false/>
							<key>STRING</key>
							<string>.cvspass</string>
							<key>TYPE</key>
							<integer>0</integer>
						</dict>
						<dict>
							<key>REGULAR_EXPRESSION</key>
							<false/>
							<key>STRING</key>
							<string>.svn</string>
							<key>TYPE</key>
							<integer>1</integer>
						</dict>
						<dict>
							<key>REGULAR_EXPRESSION</key>
							<false/>
							<key>STRING</key>
							<string>.git</string>
							<key>TYPE</key>
							<integer>1</integer>
						</dict>
						<dict>
							<key>REGULAR_EXPRESSION</key>
							<false/>
							<key>STRING</key>
							<string>.gitignore</string>
							<key>TYPE</key>
							<integer>0</integer>
						</dict>
					</array>
					<key>PROTECTED</key>
					<true/>
					<key>PROXY_NAME</key>
					<string>Remove SCM metadata</string>
					<key>PROXY_TOOLTIP</key>
					<string>Remove helper files and folders used by the CVS, SVN or Git Source Code Management systems.</string>
					<key>STATE</key>
					<true/>
				</dict>
				<dict>
					<key>PATTERNS_ARRAY</key>
					<array>
						<dict>
							<key>REGULAR_EXPRESSION</key>
							<false/>
							<key>STRING</key>
							<string>classes.nib</string>
							<key>TYPE</key>
							<integer>0</integer>
						</dict>
						<dict>
							<key>REGULAR_EXPRESSION</key>
							<false/>
							<key>STRING</key>
							<string>designable.db</string>
							<key>TYPE</key>
							<integer>0</integer>
						</dict>
						<dict>
							<key>REGULAR_EXPRESSION</key>
							<false/>
							<key>STRING</key>
							<string>info.nib</string>
							<key>TYPE</key>
							<integer>0</integer>
						</dict>
					</array>
					<key>PROTECTED</key>
					<true/>
					<key>PROXY_NAME</key>
					<string>Optimize nib files</string>
					<key>PROXY_TOOLTIP</key>
					<string>Remove "classes.nib", "info.nib" and "designable.nib" files within .nib bundles.</string>
					<key>STATE</key>
					<true/>
				</dict>
				<dict>
					<key>PATTERNS_ARRAY</key>
					<array>
						<dict>
							<key>REGULAR_EXPRESSION</key>
							<false/>
							<key>STRING</key>
							<string>Resources Disabled</string>
							<key>TYPE</key>
							<integer>1</integer>
						</dict>
					</array>
					<key>PROTECTED</key>
					<true/>
					<key>PROXY_NAME</key>
					<string>Remove Resources Disabled folders</string>
					<key>PROXY_TOOLTIP</key>
					<string>Remove "Resources Disabled" folders.</string>
					<key>STATE</key>
					<true/>
				</dict>
				<dict>
					<key>SEPARATOR</key>
					<true/>
				</dict>
			</array>
			<key>NAME</key>
			<string>OBS</string>
		</dict>
	</dict>
	<key>TYPE</key>
	<integer>0</integer>
	<key>VERSION</key>
	<integer>2</integer>
</dict>
</plist>

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>CFBundleIconFile</key>
	<string>obs.icns</string>
	<key>CFBundleName</key>
	<string>OBS</string>
	<key>CFBundleGetInfoString</key>
	<string>OBS - Free and Open Source Streaming/Recording Software</string>
	<key>CFBundleExecutable</key>
	<string>obs</string>
	<key>CFBundleIdentifier</key>
	<string>com.obsproject.obs-studio</string>
	<key>CFBundlePackageType</key>
	<string>APPL</string>
	<key>LSMinimumSystemVersion</key>
	<string>10.8.5</string>
	<key>NSHighResolutionCapable</key>
	<true/>
	<key>LSAppNapIsDisabled</key>
	<true/>
	<key>NSCameraUsageDescription</key>
 	<string>OBS needs to access the camera to enable camera sources to work.</string>
 	<key>NSMicrophoneUsageDescription</key>
 	<string>OBS needs to access the microphone to enable audio input.</string>
</dict>
</plist>

-----BEGIN PUBLIC KEY-----
MIIGPDCCBC4GByqGSM44BAEwggQhAoICAQCZZZ2y7H2GJmMfP4KQihJTJOoiGNUw
mue6sqMbH+utRykRnSKBZux6R665eRFMpNgrgFO1TLLGbdD2U31KiGtCvFJOmOl3
+QP055BuXjEG36NU7AWEFLAlbDlr/2D3oumq3Ib3iMnnr9RrVztJ2VFOvVio1eWr
ZxboVwKPK8D6BqsWiv15vbYlJnTC4Fls6ySmdjVBxwoPlTaMu1ysi5DfbIZ93s5u
aQt1FvXuWtPBWjyVUORcNbcWf49E5R2pV0OSBK95Hw2/wXz4vmj+w92dTePGnVaW
Me4CoF5PIeZILwp6DCLStX4eW2WG1NChJTC8zeQ/3bMMoGyKM/MadyvrDqMywsKY
caxkIwHrDKOEdXXGo80dIwZMMLipPA8DKhx5ojphfkeXjIhKSx+49knXT3ED5okE
Wai7tGUXj/8D8sGh+7b+AVsdujvr4v8WQaZiKUOZ2IIHOg3VLz9T9v0zet1Yt987
KNymFcp2CHeJ6KnDP/ZGQ6Nl0HsPxUgscsXV+R2FEc8Q1j0Ukkuxnopa0E4/huUu
gjyRzpXD734qFMDf7LcXca6qNjBor6gVj5sRyRKCpZ+KQfMUlr8jp506ztYSyeJu
dxJV30tQgztwkbrs02CqOt4Z3Peo6sdht7hWKSPVwmja3tq8/TfUSSoo6wKYN9/w
Mf3dVeRF8hCzJQIVAJnzuzmzQhCKPiQnl3jh5qGII2XfAoICAQCCVATAff89ceHj
ROHEbHTQFpVxJ/kRZPfxnU46DSw79Tih7tthV68oakPSOTP3cx/Tga0GwogarZ9N
F2VVan5w9OQSSewXsr5UDT5bnmJF+h+JB7TMy+sXZBYobUqjlUd5VtKc8RsN86P4
s7xbK0mA+hfe+27r18JT81/eH3xUfh7UOUGSdMN2Ch9f7RFSMZIgUAZUzu2K3ODp
hPgtc2QJ8QVAp7GLvQgw8ZUME/ChZslyBIyJvYgUIxfxlgRWYro5pQT7/ngkgdXo
wlghHKkldwMuY3zaFdhPnFNuEUEtc18ILsbz0+AnagCUd6n+3safskCRqLIHMOY6
iLBSZPX9hJQhVCqSqz1VNDDww8FNa/fojJ1Lr/TI0I+0Ib2pCiY2LChXUqGY5SLZ
2KNs5qFsyZP+I0L8YsGwqvUYyFwk7Ok224n0NtaOwqpLCrtXd/i6DaDNiaoJuwJC
1ELCfaZivorgkC5rhBt2H7qWUAR+EtrFE/gb0k/G5EIhjYql7onGbX+G2re38vQA
fg1pzguhig2dafP/BxMLZrn1Gg61xzmEYPuS9gclktaf675srv8GVb46VkOxXL+D
YvTmpJPP7UUOVlmAMCo4j4y09MW3jq9TDp42VTLeZVubyjslGnavlnq1O+ZyXUye
1FMeby65sIbSHHHwoFnRv3hLSEXI5gOCAgYAAoICAQCUkYnZkPfHfOJZI403xUYP
CE/bLpkza074Xo6EXElsWRnpQgNTx+JFOvItgj3v0OkIqDin9UredKOwfkiftslV
jxUVKA6I5kwnGvCpvTpQMLyLjq+VQr+J2D6eId6tV/iajhdu5r4JThU8KllT7Ywb
NAur34ftLNCVAMRUaDNeEoHfePgderW384e+lbvpmtifmBluammGSxxRtUsdjvJZ
BFkhaJu86CKxcU7D1lbPVOtV/jaxz6d16VdGcfBdi2LzXZzZtYpT9XGPX3NF+xii
spAURWsoe11LTRXF+eJhgCm5iIDN3kh1HEQKYKAVpmrcM0aFzk/NpS+tFyU72vaq
IRSSJw/aa1oELOAakG5oPldc4RcYWl32sbnVwXHO7TZvgTrBSC10o65MAC5CHP/s
b07heDYAIt7re7szvOYq+c/9zAMAlu3pcO8MqaXYMmybdHBXHQ2b+DdJWHmIUWcX
CbUzr09vzGkJAvqsXqbmJPr8aixrO75DhT0iDTILLWe/GWK51nf+Tg0pNxVgGyAl
BqvRqqo7SSDu9FMkwQesFFHhuoHLyEHwVPJ+sMQTNwQcm9c6YuW8EYDRSkeKLWYk
3fkjG+Pe9uVE8a1taDg3FjSY0UqjUT6XMw+i0Lajyus2L6wFBwrrGM6E4xa6x1CC
MGjmuSOlPA1umQsToIcO4g==
-----END PUBLIC KEY-----

CI/install/osx/SyphonInject.pkg
CI/install/osx/background.png
CI/install/osx/background.pxd/QuickLook/Icon.tiff
CI/install/osx/background.pxd/QuickLook/Preview.tiff
CI/install/osx/background.pxd/QuickLook/Thumbnail.tiff
CI/install/osx/background.pxd/data/556CF265-5721-4F18-BE83-8CF39483B4C2
CI/install/osx/background.pxd/data/8CA689C3-ED2A-459E-952C-E08026CFCD07
CI/install/osx/background.pxd/metadata.info
CI/install/osx/background.tiff
CI/install/osx/background@2x.png

dmgbuild -s ./settings.json "OBS" obs.dmg

#!/usr/bin/env python

candidate_paths = "bin obs-plugins data".split()

plist_path = "../cmake/osxbundle/Info.plist"
icon_path = "../cmake/osxbundle/obs.icns"
run_path = "../cmake/osxbundle/obslaunch.sh"

#not copied
blacklist = """/usr /System""".split()

#copied
whitelist = """/usr/local""".split()

#
#
#


from sys import argv
from glob import glob
from subprocess import check_output, call
from collections import namedtuple
from shutil import copy, copytree, rmtree
from os import makedirs, rename, walk, path as ospath
import plistlib

import argparse

def _str_to_bool(s):
    """Convert string to bool (in argparse context)."""
    if s.lower() not in ['true', 'false']:
        raise ValueError('Need bool; got %r' % s)
    return {'true': True, 'false': False}[s.lower()]

def add_boolean_argument(parser, name, default=False):
    """Add a boolean argument to an ArgumentParser instance."""
    group = parser.add_mutually_exclusive_group()
    group.add_argument(
        '--' + name, nargs='?', default=default, const=True, type=_str_to_bool)
    group.add_argument('--no' + name, dest=name, action='store_false')

parser = argparse.ArgumentParser(description='obs-studio package util')
parser.add_argument('-d', '--base-dir', dest='dir', default='rundir/RelWithDebInfo')
parser.add_argument('-n', '--build-number', dest='build_number', default='0')
parser.add_argument('-k', '--public-key', dest='public_key', default='OBSPublicDSAKey.pem')
parser.add_argument('-f', '--sparkle-framework', dest='sparkle', default=None)
parser.add_argument('-b', '--base-url', dest='base_url', default='https://obsproject.com/osx_update')
parser.add_argument('-u', '--user', dest='user', default='jp9000')
parser.add_argument('-c', '--channel', dest='channel', default='master')
add_boolean_argument(parser, 'stable', default=False)
parser.add_argument('-p', '--prefix', dest='prefix', default='')
args = parser.parse_args()

def cmd(cmd):
    import subprocess
    import shlex
    return subprocess.check_output(shlex.split(cmd)).rstrip('\r\n')

LibTarget = namedtuple("LibTarget", ("path", "external", "copy_as"))

inspect = list()

inspected = set()

build_path = args.dir
build_path = build_path.replace("\\ ", " ")

def add(name, external=False, copy_as=None):
	if external and copy_as is None:
		copy_as = name.split("/")[-1]
	if name[0] != "/":
		name = build_path+"/"+name
	t = LibTarget(name, external, copy_as)
	if t in inspected:
		return
	inspect.append(t)
	inspected.add(t)


for i in candidate_paths:
	print("Checking " + i)
	for root, dirs, files in walk(build_path+"/"+i):
		for file_ in files:
			if ".ini" in file_:
				continue
			if ".png" in file_:
				continue
			if ".effect" in file_:
				continue
			if ".py" in file_:
				continue
			if ".json" in file_:
				continue
			path = root + "/" + file_
			try:
				out = check_output("{0}otool -L '{1}'".format(args.prefix, path), shell=True,
						universal_newlines=True)
				if "is not an object file" in out:
					continue
			except:
				continue
			rel_path = path[len(build_path)+1:]
			print(repr(path), repr(rel_path))
			add(rel_path)

def add_plugins(path, replace):
	for img in glob(path.replace(
		"lib/QtCore.framework/Versions/5/QtCore",
		"plugins/%s/*"%replace).replace(
			"Library/Frameworks/QtCore.framework/Versions/5/QtCore",
			"share/qt5/plugins/%s/*"%replace)):
		if "_debug" in img:
			continue
		add(img, True, img.split("plugins/")[-1])

actual_sparkle_path = '@loader_path/Frameworks/Sparkle.framework/Versions/A/Sparkle'

while inspect:
	target = inspect.pop()
	print("inspecting", repr(target))
	path = target.path
	if path[0] == "@":
		continue
	out = check_output("{0}otool -L '{1}'".format(args.prefix, path), shell=True,
			universal_newlines=True)

	if "QtCore" in path:
		add_plugins(path, "platforms")
		add_plugins(path, "imageformats")
		add_plugins(path, "accessible")
		add_plugins(path, "styles")


	for line in out.split("\n")[1:]:
		new = line.strip().split(" (")[0]
		if '@' in new and "sparkle.framework" in new.lower():
			actual_sparkle_path = new
			print "Using sparkle path:", repr(actual_sparkle_path)
		if not new or new[0] == "@" or new.endswith(path.split("/")[-1]):
			continue
		whitelisted = False
		for i in whitelist:
			if new.startswith(i):
				whitelisted = True
		if not whitelisted:
			blacklisted = False
			for i in blacklist:
				if new.startswith(i):
					blacklisted = True
					break
			if blacklisted:
				continue
		add(new, True)

changes = list()
for path, external, copy_as in inspected:
	if not external:
		continue #built with install_rpath hopefully
	changes.append("-change '%s' '@rpath/%s'"%(path, copy_as))
changes = " ".join(changes)

info = plistlib.readPlist(plist_path)

latest_tag = cmd('git describe --tags --abbrev=0')
log = cmd('git log --pretty=oneline {0}...HEAD'.format(latest_tag))

from os import path
# set version
if args.stable:
    info["CFBundleVersion"] = latest_tag
    info["CFBundleShortVersionString"] = latest_tag
    info["SUFeedURL"] = '{0}/stable/updates.xml'.format(args.base_url)
else:
    info["CFBundleVersion"] = args.build_number
    info["CFBundleShortVersionString"] = '{0}.{1}'.format(latest_tag, args.build_number)
    info["SUFeedURL"] = '{0}/{1}/{2}/updates.xml'.format(args.base_url, args.user, args.channel)

info["SUPublicDSAKeyFile"] = path.basename(args.public_key)
info["OBSFeedsURL"] = '{0}/feeds.xml'.format(args.base_url)

app_name = info["CFBundleName"]+".app"
icon_file = "tmp/Contents/Resources/%s"%info["CFBundleIconFile"]

copytree(build_path, "tmp/Contents/Resources/", symlinks=True)
copy(icon_path, icon_file)
plistlib.writePlist(info, "tmp/Contents/Info.plist")
makedirs("tmp/Contents/MacOS")
copy(run_path, "tmp/Contents/MacOS/%s"%info["CFBundleExecutable"])
try:
	copy(args.public_key, "tmp/Contents/Resources")
except:
	pass

if args.sparkle is not None:
    copytree(args.sparkle, "tmp/Contents/Frameworks/Sparkle.framework", symlinks=True)

prefix = "tmp/Contents/Resources/"
sparkle_path = '@loader_path/{0}/Frameworks/Sparkle.framework/Versions/A/Sparkle'

cmd('{0}install_name_tool -change {1} {2} {3}/bin/obs'.format(
    args.prefix, actual_sparkle_path, sparkle_path.format('../..'), prefix))



for path, external, copy_as in inspected:
	id_ = ""
	filename = path
	rpath = ""
	if external:
		if copy_as == "Python":
			continue
		id_ = "-id '@rpath/%s'"%copy_as
		filename = prefix + "bin/" +copy_as
		rpath = "-add_rpath @loader_path/ -add_rpath @executable_path/"
		if "/" in copy_as:
			try:
				dirs = copy_as.rsplit("/", 1)[0]
				makedirs(prefix + "bin/" + dirs)
			except:
				pass
		copy(path, filename)
	else:
		filename = path[len(build_path)+1:]
		id_ = "-id '@rpath/../%s'"%filename
		if not filename.startswith("bin"):
			print(filename)
			rpath = "-add_rpath '@loader_path/{}/'".format(ospath.relpath("bin/", ospath.dirname(filename)))
		filename = prefix + filename

	cmd = "{0}install_name_tool {1} {2} {3} '{4}'".format(args.prefix, changes, id_, rpath, filename)
	call(cmd, shell=True)

try:
	rename("tmp", app_name)
except:
	print("App already exists")
	rmtree("tmp")

CI/install/osx/dylibBundler

tiffutil -cathidpicheck background.png background@2x.png -out background.tiff

CI/install/osx/obs.icns
CI/install/osx/obs.png

# Exit if something fails
set -e

rm -rf ./OBS.app

mkdir OBS.app
mkdir OBS.app/Contents
mkdir OBS.app/Contents/MacOS
mkdir OBS.app/Contents/PlugIns
mkdir OBS.app/Contents/Resources

cp -R rundir/RelWithDebInfo/bin/ ./OBS.app/Contents/MacOS
cp -R rundir/RelWithDebInfo/data ./OBS.app/Contents/Resources
cp ../CI/install/osx/obs.icns ./OBS.app/Contents/Resources
cp -R rundir/RelWithDebInfo/obs-plugins/ ./OBS.app/Contents/PlugIns
cp ../CI/install/osx/Info.plist ./OBS.app/Contents

../CI/install/osx/dylibBundler -b -cd -d ./OBS.app/Contents/Frameworks -p @executable_path/../Frameworks/ \
-s ./OBS.app/Contents/MacOS \
-s /usr/local/opt/mbedtls/lib/ \
-x ./OBS.app/Contents/PlugIns/coreaudio-encoder.so \
-x ./OBS.app/Contents/PlugIns/decklink-ouput-ui.so \
-x ./OBS.app/Contents/PlugIns/frontend-tools.so \
-x ./OBS.app/Contents/PlugIns/image-source.so \
-x ./OBS.app/Contents/PlugIns/linux-jack.so \
-x ./OBS.app/Contents/PlugIns/mac-avcapture.so \
-x ./OBS.app/Contents/PlugIns/mac-capture.so \
-x ./OBS.app/Contents/PlugIns/mac-decklink.so \
-x ./OBS.app/Contents/PlugIns/mac-syphon.so \
-x ./OBS.app/Contents/PlugIns/mac-vth264.so \
-x ./OBS.app/Contents/PlugIns/obs-browser.so \
-x ./OBS.app/Contents/PlugIns/obs-browser-page \
-x ./OBS.app/Contents/PlugIns/obs-ffmpeg.so \
-x ./OBS.app/Contents/PlugIns/obs-filters.so \
-x ./OBS.app/Contents/PlugIns/obs-transitions.so \
-x ./OBS.app/Contents/PlugIns/obs-vst.so \
-x ./OBS.app/Contents/PlugIns/rtmp-services.so \
-x ./OBS.app/Contents/MacOS/obs \
-x ./OBS.app/Contents/MacOS/obs-ffmpeg-mux \
-x ./OBS.app/Contents/MacOS/obslua.so \
-x ./OBS.app/Contents/MacOS/_obspython.so \
-x ./OBS.app/Contents/PlugIns/obs-x264.so \
-x ./OBS.app/Contents/PlugIns/text-freetype2.so \
-x ./OBS.app/Contents/PlugIns/obs-libfdk.so
# -x ./OBS.app/Contents/PlugIns/obs-outputs.so \

/usr/local/Cellar/qt/5.14.1/bin/macdeployqt ./OBS.app

mv ./OBS.app/Contents/MacOS/libobs-opengl.so ./OBS.app/Contents/Frameworks

rm -f -r ./OBS.app/Contents/Frameworks/QtNetwork.framework

# put qt network in here becasuse streamdeck uses it
cp -R /usr/local/opt/qt/lib/QtNetwork.framework ./OBS.app/Contents/Frameworks
chmod -R +w ./OBS.app/Contents/Frameworks/QtNetwork.framework
rm -r ./OBS.app/Contents/Frameworks/QtNetwork.framework/Headers
rm -r ./OBS.app/Contents/Frameworks/QtNetwork.framework/Versions/5/Headers/
chmod 644 ./OBS.app/Contents/Frameworks/QtNetwork.framework/Versions/5/Resources/Info.plist
install_name_tool -id @executable_path/../Frameworks/QtNetwork.framework/Versions/5/QtNetwork ./OBS.app/Contents/Frameworks/QtNetwork.framework/Versions/5/QtNetwork
install_name_tool -change /usr/local/Cellar/qt/5.14.1/lib/QtCore.framework/Versions/5/QtCore @executable_path/../Frameworks/QtCore.framework/Versions/5/QtCore ./OBS.app/Contents/Frameworks/QtNetwork.framework/Versions/5/QtNetwork


# decklink ui qt
install_name_tool -change /usr/local/opt/qt/lib/QtGui.framework/Versions/5/QtGui @executable_path/../Frameworks/QtGui.framework/Versions/5/QtGui ./OBS.app/Contents/PlugIns/decklink-ouput-ui.so
install_name_tool -change /usr/local/opt/qt/lib/QtCore.framework/Versions/5/QtCore @executable_path/../Frameworks/QtCore.framework/Versions/5/QtCore ./OBS.app/Contents/PlugIns/decklink-ouput-ui.so
install_name_tool -change /usr/local/opt/qt/lib/QtWidgets.framework/Versions/5/QtWidgets @executable_path/../Frameworks/QtWidgets.framework/Versions/5/QtWidgets ./OBS.app/Contents/PlugIns/decklink-ouput-ui.so

# frontend tools qt
install_name_tool -change /usr/local/opt/qt/lib/QtGui.framework/Versions/5/QtGui @executable_path/../Frameworks/QtGui.framework/Versions/5/QtGui ./OBS.app/Contents/PlugIns/frontend-tools.so
install_name_tool -change /usr/local/opt/qt/lib/QtCore.framework/Versions/5/QtCore @executable_path/../Frameworks/QtCore.framework/Versions/5/QtCore ./OBS.app/Contents/PlugIns/frontend-tools.so
install_name_tool -change /usr/local/opt/qt/lib/QtWidgets.framework/Versions/5/QtWidgets @executable_path/../Frameworks/QtWidgets.framework/Versions/5/QtWidgets ./OBS.app/Contents/PlugIns/frontend-tools.so

# vst qt
install_name_tool -change /usr/local/opt/qt/lib/QtGui.framework/Versions/5/QtGui @executable_path/../Frameworks/QtGui.framework/Versions/5/QtGui ./OBS.app/Contents/PlugIns/obs-vst.so
install_name_tool -change /usr/local/opt/qt/lib/QtCore.framework/Versions/5/QtCore @executable_path/../Frameworks/QtCore.framework/Versions/5/QtCore ./OBS.app/Contents/PlugIns/obs-vst.so
install_name_tool -change /usr/local/opt/qt/lib/QtWidgets.framework/Versions/5/QtWidgets @executable_path/../Frameworks/QtWidgets.framework/Versions/5/QtWidgets ./OBS.app/Contents/PlugIns/obs-vst.so
install_name_tool -change /usr/local/opt/qt/lib/QtMacExtras.framework/Versions/5/QtMacExtras @executable_path/../Frameworks/QtMacExtras.framework/Versions/5/QtMacExtras ./OBS.app/Contents/PlugIns/obs-vst.so

def cmd(cmd):
    import subprocess
    import shlex
    return subprocess.check_output(shlex.split(cmd)).rstrip('\r\n')

def get_tag_info(tag):
    rev = cmd('git rev-parse {0}'.format(latest_tag))
    anno = cmd('git cat-file -p {0}'.format(rev))
    tag_info = []
    for i, v in enumerate(anno.splitlines()):
        if i <= 4:
            continue
        tag_info.append(v.lstrip())

    return tag_info

def gen_html(github_user, latest_tag):

    url = 'https://github.com/{0}/obs-studio/commit/%H'.format(github_user)

    with open('readme.html', 'w') as f:
        f.write("<html><body>")
        log_cmd = """git log {0}...HEAD --pretty=format:'<li>&bull; <a href="{1}">(view)</a> %s</li>'"""
        log_res = cmd(log_cmd.format(latest_tag, url))
        if len(log_res.splitlines()):
            f.write('<p>Changes since {0}: (Newest to oldest)</p>'.format(latest_tag))
            f.write(log_res)

        ul = False
        f.write('<p>')
        import re

        for l in get_tag_info(latest_tag):
            if not len(l):
                continue
            if l.startswith('*'):
                ul = True
                if not ul:
                    f.write('<ul>')
                f.write('<li>&bull; {0}</li>'.format(re.sub(r'^(\s*)?[*](\s*)?', '', l)))
            else:
                if ul:
                    f.write('</ul><p/>')
                ul = False
                f.write('<p>{0}</p>'.format(l))
        if ul:
            f.write('</ul>')
        f.write('</p></body></html>')

    cmd('textutil -convert rtf readme.html -output readme.rtf')
    cmd("""sed -i '' 's/Times-Roman/Verdana/g' readme.rtf""")

def save_manifest(latest_tag, user, jenkins_build, branch, stable):
    log = cmd('git log --pretty=oneline {0}...HEAD'.format(latest_tag))
    manifest = {}
    manifest['commits'] = []
    for v in log.splitlines():
        manifest['commits'].append(v)
    manifest['tag'] = {
        'name': latest_tag,
        'description': get_tag_info(latest_tag)
    }

    manifest['version'] = cmd('git rev-list HEAD --count')
    manifest['sha1'] = cmd('git rev-parse HEAD')
    manifest['jenkins_build'] = jenkins_build

    manifest['user'] = user
    manifest['branch'] = branch
    manifest['stable'] = stable

    import cPickle
    with open('manifest', 'w') as f:
        cPickle.dump(manifest, f)

def prepare_pkg(project, package_id):
    cmd('packagesutil --file "{0}" set package-1 identifier {1}'.format(project, package_id))
    cmd('packagesutil --file "{0}" set package-1 version {1}'.format(project, '1.0'))


import argparse
parser = argparse.ArgumentParser(description='obs-studio package util')
parser.add_argument('-u', '--user', dest='user', default='jp9000')
parser.add_argument('-p', '--package-id', dest='package_id', default='org.obsproject.pkg.obs-studio')
parser.add_argument('-f', '--project-file', dest='project', default='OBS.pkgproj')
parser.add_argument('-j', '--jenkins-build', dest='jenkins_build', default='0')
parser.add_argument('-b', '--branch', dest='branch', default='master')
parser.add_argument('-s', '--stable', dest='stable', required=False, action='store_true', default=False)
args = parser.parse_args()

latest_tag = cmd('git describe --tags --abbrev=0')
gen_html(args.user, latest_tag)
prepare_pkg(args.project, args.package_id)
save_manifest(latest_tag, args.user, args.jenkins_build, args.branch, args.stable)

#!/usr/bin/env bash

{
    "title": "OBS",
    "background": "../CI/install/osx/background.tiff",
    "format": "UDZO",
    "compression-level": 9,
    "window": { "position": { "x": 100, "y": 100 },
                "size": { "width": 540, "height": 380 } },
    "contents": [
        { "x": 120, "y": 180, "type": "file",
          "path": "./OBS.app" },
        { "x": 420, "y": 180, "type": "link", "path": "/Applications" }
    ]
}

CI/osxcert/Certificates.p12.enc

brew "jack"
brew "speexdsp"
brew "cmake"
brew "freetype"
brew "fdk-aac"
brew "https://gist.githubusercontent.com/DDRBoxman/9c7a2b08933166f4b61ed9a44b242609/raw/ef4de6c587c6bd7f50210eccd5bd51ff08e6de13/qt.rb", link: true
brew "https://gist.githubusercontent.com/DDRBoxman/4cada55c51803a2f963fa40ce55c9d3e/raw/572c67e908bfbc1bcb8c476ea77ea3935133f5b5/swig.rb", link: true

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>CFBundleIconFile</key>
	<string>obs.icns</string>
	<key>CFBundleName</key>
	<string>OBS</string>
	<key>CFBundleGetInfoString</key>
	<string>OBS - Free and Open Source Streaming/Recording Software</string>
	<key>CFBundleExecutable</key>
	<string>obs</string>
	<key>CFBundleIdentifier</key>
	<string>com.obsproject.obs-studio</string>
	<key>CFBundlePackageType</key>
	<string>APPL</string>
	<key>LSMinimumSystemVersion</key>
	<string>10.8.5</string>
	<key>NSHighResolutionCapable</key>
	<true/>
	<key>LSAppNapIsDisabled</key>
	<true/>
	<key>NSCameraUsageDescription</key>
 	<string>OBS needs to access the camera to enable camera sources to work.</string>
 	<key>NSMicrophoneUsageDescription</key>
 	<string>OBS needs to access the microphone to enable audio input.</string>
</dict>
</plist>

-----BEGIN PUBLIC KEY-----
MIIGPDCCBC4GByqGSM44BAEwggQhAoICAQCZZZ2y7H2GJmMfP4KQihJTJOoiGNUw
mue6sqMbH+utRykRnSKBZux6R665eRFMpNgrgFO1TLLGbdD2U31KiGtCvFJOmOl3
+QP055BuXjEG36NU7AWEFLAlbDlr/2D3oumq3Ib3iMnnr9RrVztJ2VFOvVio1eWr
ZxboVwKPK8D6BqsWiv15vbYlJnTC4Fls6ySmdjVBxwoPlTaMu1ysi5DfbIZ93s5u
aQt1FvXuWtPBWjyVUORcNbcWf49E5R2pV0OSBK95Hw2/wXz4vmj+w92dTePGnVaW
Me4CoF5PIeZILwp6DCLStX4eW2WG1NChJTC8zeQ/3bMMoGyKM/MadyvrDqMywsKY
caxkIwHrDKOEdXXGo80dIwZMMLipPA8DKhx5ojphfkeXjIhKSx+49knXT3ED5okE
Wai7tGUXj/8D8sGh+7b+AVsdujvr4v8WQaZiKUOZ2IIHOg3VLz9T9v0zet1Yt987
KNymFcp2CHeJ6KnDP/ZGQ6Nl0HsPxUgscsXV+R2FEc8Q1j0Ukkuxnopa0E4/huUu
gjyRzpXD734qFMDf7LcXca6qNjBor6gVj5sRyRKCpZ+KQfMUlr8jp506ztYSyeJu
dxJV30tQgztwkbrs02CqOt4Z3Peo6sdht7hWKSPVwmja3tq8/TfUSSoo6wKYN9/w
Mf3dVeRF8hCzJQIVAJnzuzmzQhCKPiQnl3jh5qGII2XfAoICAQCCVATAff89ceHj
ROHEbHTQFpVxJ/kRZPfxnU46DSw79Tih7tthV68oakPSOTP3cx/Tga0GwogarZ9N
F2VVan5w9OQSSewXsr5UDT5bnmJF+h+JB7TMy+sXZBYobUqjlUd5VtKc8RsN86P4
s7xbK0mA+hfe+27r18JT81/eH3xUfh7UOUGSdMN2Ch9f7RFSMZIgUAZUzu2K3ODp
hPgtc2QJ8QVAp7GLvQgw8ZUME/ChZslyBIyJvYgUIxfxlgRWYro5pQT7/ngkgdXo
wlghHKkldwMuY3zaFdhPnFNuEUEtc18ILsbz0+AnagCUd6n+3safskCRqLIHMOY6
iLBSZPX9hJQhVCqSqz1VNDDww8FNa/fojJ1Lr/TI0I+0Ib2pCiY2LChXUqGY5SLZ
2KNs5qFsyZP+I0L8YsGwqvUYyFwk7Ok224n0NtaOwqpLCrtXd/i6DaDNiaoJuwJC
1ELCfaZivorgkC5rhBt2H7qWUAR+EtrFE/gb0k/G5EIhjYql7onGbX+G2re38vQA
fg1pzguhig2dafP/BxMLZrn1Gg61xzmEYPuS9gclktaf675srv8GVb46VkOxXL+D
YvTmpJPP7UUOVlmAMCo4j4y09MW3jq9TDp42VTLeZVubyjslGnavlnq1O+ZyXUye
1FMeby65sIbSHHHwoFnRv3hLSEXI5gOCAgYAAoICAQCUkYnZkPfHfOJZI403xUYP
CE/bLpkza074Xo6EXElsWRnpQgNTx+JFOvItgj3v0OkIqDin9UredKOwfkiftslV
jxUVKA6I5kwnGvCpvTpQMLyLjq+VQr+J2D6eId6tV/iajhdu5r4JThU8KllT7Ywb
NAur34ftLNCVAMRUaDNeEoHfePgderW384e+lbvpmtifmBluammGSxxRtUsdjvJZ
BFkhaJu86CKxcU7D1lbPVOtV/jaxz6d16VdGcfBdi2LzXZzZtYpT9XGPX3NF+xii
spAURWsoe11LTRXF+eJhgCm5iIDN3kh1HEQKYKAVpmrcM0aFzk/NpS+tFyU72vaq
IRSSJw/aa1oELOAakG5oPldc4RcYWl32sbnVwXHO7TZvgTrBSC10o65MAC5CHP/s
b07heDYAIt7re7szvOYq+c/9zAMAlu3pcO8MqaXYMmybdHBXHQ2b+DdJWHmIUWcX
CbUzr09vzGkJAvqsXqbmJPr8aixrO75DhT0iDTILLWe/GWK51nf+Tg0pNxVgGyAl
BqvRqqo7SSDu9FMkwQesFFHhuoHLyEHwVPJ+sMQTNwQcm9c6YuW8EYDRSkeKLWYk
3fkjG+Pe9uVE8a1taDg3FjSY0UqjUT6XMw+i0Lajyus2L6wFBwrrGM6E4xa6x1CC
MGjmuSOlPA1umQsToIcO4g==
-----END PUBLIC KEY-----

CI/scripts/macos/app/dylibbundler
