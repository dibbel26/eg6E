_game="eg6E"
_server_root="/srv/${_game}"
#_conf_dir="/bin/conf.d/${_game}d"

pkgname="${_game}-server"
pkgver=1.16.5
pkgrel=1
_eg_ver=1.8.0
_mng_ver=1.0.2
pkgdesc="Server-Wrapper for mineraft Enigmatica6E"
arch=('any')
url="https://minecraft.net/"
license=('custom')
depends=('java-runtime-headless=8' 'tmux' 'sudo' 'bash' 'awk' 'sed')
optdepends=("tar: needed in order to create world backups"
            "netcat: required in order to suspend an idle server")
makedepends=('unzip')
backup=('etc/conf.d/${_game}')
#install="${pkgname}.install"
source=("minecraft-server-${_mng_ver}.tar.gz"::"https://github.com/Edenhofer/minecraft-server/archive/refs/tags/v${_mng_ver}.tar.gz"
        "Enigmatica6ExpertServer-${_eg_ver}.zip"::"https://mediafilez.forgecdn.net/files/4437/745/Enigmatica6ExpertServer-1.8.0.zip")
noextract=("Enigmatica6ExpertServer-${_eg_ver}.zip")
sha512sums=('11d708d511b63e5541bcc1dbcaf29abbf7cb9583b1d313028770a39b26b41d48dcba023f7e1d6fe30f3c093d20e10a43363011edd432e5785a4580e5c5f852a6'
            '03560e914d181955fa68a1f603c66fc9985760d6215948555da90c4df37be1c251799cc4ee4f14927aa80f73574071b2d6e13ceeab6b53a7080f9bb3d956fbfa')

prepare() {
  cd ${srcdir}
  unzip "Enigmatica6ExpertServer-${_eg_ver}.zip" -d "Enigmatica6ExpertServer-${_eg_ver}"
  #unzip -u "Enigmatica6ExpertServer-${_eg_ver}.zip" -d "Enigmatica6ExpertServer-${_eg_ver}"
  cd "Enigmatica6ExpertServer-${_eg_ver}"
  echo "eula=true" > eula.txt
  sed -i '/java -jar serverstarter-2.4.0.jar/c\java -jar serverstarter-2.4.0.jar install' start-server.sh

  #args from original serverstarter TODO: Automate arg-extraction
  echo -XX:+UseG1GC > unix_args.txt
  echo -XX:+ParallelRefProcEnabled >> unix_args.txt
  echo -XX:MaxGCPauseMillis=200 >> unix_args.txt
  echo -XX:+UnlockExperimentalVMOptions >> unix_args.txt
  echo -XX:+DisableExplicitGC >> unix_args.txt
  echo -XX:+AlwaysPreTouch >> unix_args.txt
  echo -XX:G1NewSizePercent=30 >> unix_args.txt
  echo -XX:G1MaxNewSizePercent=40 >> unix_args.txt
  echo -XX:G1HeapRegionSize=8M >> unix_args.txt
  echo -XX:G1ReservePercent=20 >> unix_args.txt
  echo -XX:G1HeapWastePercent=5 >> unix_args.txt
  echo -XX:G1MixedGCCountTarget=4 >> unix_args.txt
  echo -XX:InitiatingHeapOccupancyPercent=15 >> unix_args.txt
  echo -XX:G1MixedGCLiveThresholdPercent=90 >> unix_args.txt
  echo -XX:G1RSetUpdatingPauseTimePercent=5 >> unix_args.txt
  echo -XX:SurvivorRatio=32 >> unix_args.txt
  echo -XX:+PerfDisableSharedMem >> unix_args.txt
  echo -XX:MaxTenuringThreshold=1 >> unix_args.txt
  echo -Dusing.aikars.flags=https://mcflags.emc.gs >> unix_args.txt
  echo -Daikars.new.flags=true >> unix_args.txt
  echo -Dfml.readTimeout=180 >> unix_args.txt # servertimeout
  echo -Dfml.queryResult=confirm >> unix_args.txt # auto /fmlconfirm
#  echo --add-opens=java.base/sun.security.util=ALL-UNNAMED  >> unix_args.txt #java16+ support
#  echo --add-opens=java.base/java.util.jar=ALL-UNNAMED  >> unix_args.txt # java16+ support
#  echo -XX:+IgnoreUnrecognizedVMOptions  >> unix_args.txt # java16+ support
  
}

build() {
  cd ${srcdir}/Enigmatica6ExpertServer-${_eg_ver}
  bash ./start-server.sh
  echo "#By changing the setting below to TRUE you are indicating your agreement to our EULA (https://account.mojang.com/documents/minecraft_eula)." > eula.txt
  echo "$(date +%c)" >> eula.txt
  # echo "#Sat Apr 15 03:36:55 GMT+2 2023" >> eula.txt
  echo "eula=false" >> eula.txt
  cd ${srcdir}

  make -C "${srcdir}/minecraft-server-${_mng_ver}" clean

  make -C "${srcdir}/minecraft-server-${_mng_ver}" \
    GAME=${_game} \
    INAME=${_game}d \
    SERVER_ROOT=${_server_root} \
    BACKUP_PATHS="world" \
    GAME_USER=${_game} \
    MAIN_EXECUTABLE="${_game}_server.jar" \
    SERVER_START_CMD="java -Xms4096M -Xmx8192M @${_server_root}/unix_args.txt -jar ./${_game}_server.jar nogui" \
    all
}

#/usr/lib/jvm/java-11-openjdk/jre/bin/
#SERVER_START_CMD="/usr/lib/jvm/java-8-openjdk/jre/bin/java @user_jvm_args.txt @unix_args.txt jar './${MAIN_EXECUTABLE}' nogui" \
#SERVER_START_CMD="/usr/lib/jvm/java-8-openjdk/jre/bin/java -Xms512M -Xms1024M @unix_args.txt -jar ./${_game}_server.jar nogui" \ 
#might be preferable since RAM Limits should only me adjustable with root rights in /bin/conf.d/${_game}d
#SERVER_START_CMD="/usr/lib/jvm/java-8-openjdk/jre/bin/java -Xms512M -Xms1024M -jar ./${_game}_server.jar nogui" \

package() {
  make -C "${srcdir}/minecraft-server-${_mng_ver}" \
      DESTDIR="${pkgdir}" \
      GAME=${_game} \
      INAME=${_game}d \
      install

  echo "installing to ${_server_root}..."
  cd ${srcdir}/Enigmatica6ExpertServer-${_eg_ver}
  find ./ -type f \
  -not -path "./server-guide.txt" \
  -not -path "./installer.jar.log" \
  -not -path "./start-server.bat" \
  -not -path "./start-server.sh" \
  -not -path "./serverstarter-2.4.0.jar" \
  -not -path "./serverstarter.lock" \
  -not -path "./modpack-download.zip" \
  -not -path "./server-setup-config.yaml" \
  -print0 | xargs -0 -i@ install -o ${_game} -g ${_game} -Dm664 "@" "${pkgdir}${_server_root}/@"
  #-print0 | xargs -0 -i@ install -o ${_game} -g ${_game} -Dm644 "@" "${pkgdir}${_server_root}/@"
  
  chown -R ${_game}:${_game} ${pkgdir}${_server_root}
  chmod g+rw -R ${pkgdir}${_server_root}

  echo "linking main executables..."
  ln -s "forge-${pkgver}-36.2.39.jar" "${pkgdir}${_server_root}/${_game}_server.jar"

  echo "linking logs..."
  mkdir -p "${pkgdir}/var/log/"
	install -dm2755 "${pkgdir}/${_server_root}/logs"
	ln -s "${_server_root}/logs" "${pkgdir}/var/log/${_game}"
  echo "done"
  echo "don't forget to accept eula.txt !"
}
