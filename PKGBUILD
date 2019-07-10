# Maintainer: Vladimir Demenev <vademenev@gmail.com>
# NOTE: https://wiki.archlinux.org/index.php/VMware
# 
# For compile package run:
#   $ updpkgsums && makepkg -fs
# 
# Upgrade or add package into system:
#   $ sudo pacman -U package_name.pkg.tar.xz
# 
# Remove package from system:
#   sudo pacman -Rn package_name
# 
#-------------------------------------------------------------------
# FIXME: Проверить куда копируются конфиги старта кернел модулей. 
# Попытаться убрать их из /lib/modprobe.d, /lib/modules-load.d и перенести в
# /etc/modprobe.d и соответственно в /etc/modules-load.d


pkgname=vmware-player
pkgdesc='VMware Player'
pkgver=15.1.0
pkgbuild=13591040
pkgrel=2
arch=('x86_64')
url='https://www.vmware.com/products/workstation-player.html'
license=('custom: commercial')

# Pre-Post installation scripts for pacman: https://wiki.archlinux.org/index.php/PKGBUILD#install
install="pre-post-scripts.install"

#-------------------------------------------------------------------

depends=(
    fuse2                     # for vmware-vmblock-fuse
    gtkmm3                    # for the GUI
    libcanberra               # for event sounds
    pcsclite
    gtk3                      # It need for using Arch GTK3 library (for theme integration)
    gcr
    linux-headers             # It need for kernel modules compilation (vmmon, vmnet)
)
optdepends=()
makedepends=(sqlite)
conflicts=(
    vmware-player
    vmware-player-bin
    vmware-workstation-bin
    vmware-workstation
    vmware-modules-dkms
    vmware-ovftool
    vmware-patch
    vmware-systemd-services
)
backup=(
  'etc/vmware/config'
)
#-------------------------------------------------------------------

source_x86_64=(
    "VMware-Player-${pkgver}-${pkgbuild}.${CARCH}.bundle::https://download3.vmware.com/software/player/file/VMware-Player-${pkgver}-${pkgbuild}.${CARCH}.bundle"
    vmware-configure-initscript.sh
    vmware-networks-configuration.service
    vmware-networks.service
    vmware-usbarbitrator.service
    vmware-bootstrap
    vmware-initd                     # Installing to /usr/lib/systemd/scripts/vmware
    vmware-config                    # Main file configuration, installing to /etc/vmware/config
    vmware-modules-load.conf         # Config file of auto load kernel modules. Installing to /usr/lib/modules-load.d/vmware.conf
)
# For update/generate MD5 sum for source files, run `updpkgsums`
md5sums_x86_64=('02580e483bf7034641269696838b3d33'
                'b691e490f36ec574cfb301873deb6b1c'
                '5a4d539f587486f3b778eaebd76e780e'
                'fcd2864ee465b1925592c65eb8e65ca3'
                '5a64ef908a36f1ce2638ed08fd7d12fb'
                '9d2c6433034063b0f1d5bbd415200b4a'
                'eda04a578c729b4177d65e3b3f4f1fff'
                '1722b8b6ff78ac1b185a5e34517f94d6'
                'e535a198f2eae87c2446aa38c6006385')

#-------------------------------------------------------------------
patch_host_modules() {
    # INFO: https://github.com/mkubecek/vmware-host-modules/blob/player-15.1.0/INSTALL
    patch_dir="${srcdir}/patch"
    install -d -m 755 "${patch_dir}"
    cd "${patch_dir}"
    git clone https://github.com/mkubecek/vmware-host-modules.git
    cd vmware-host-modules
    git checkout "player-${pkgver}"
    tar -cf ${srcdir}/extracted/vmware-vmx/lib/modules/source/vmmon.tar vmmon-only
    tar -cf ${srcdir}/extracted/vmware-vmx/lib/modules/source/vmnet.tar vmnet-only
}

prepare() {
    # VMware-Player-*.bundle is a bash executable script (binary data).
    # You can verify this with the `file` utility: $ file VMware-Player-*.bundle

    extracted_dir="${srcdir}/extracted"
    [[ -d "$extracted_dir" ]] && rm -r "$extracted_dir"
    bash "$(readlink -f "${srcdir}/VMware-Player-${pkgver}-${pkgbuild}.${CARCH}.bundle")" --extract "$extracted_dir"

    #################### Patch original files
    kernel_version="$(uname -r | cut -d '-' -f 1)"

    if [[ "${pkgver}" > 14.999 && "${pkgver}" < 15.0.999 ]] && [[ "${kernel_version}" > 4.999 ]]; then
        patch_host_modules
    elif [[ "${pkgver}" > 15.0.999 && "${pkgver}" < 15.1.999 ]] && [[ "${kernel_version}" > 5.0.999 ]]; then
        patch_host_modules
    fi;
}

package() {

    local vmware_installer_version=$(grep -oPm1 "(?<=<version>)[^<]+" "${srcdir}/extracted/vmware-installer/manifest.xml")
    local vmware_usbarbitrator_version=$(grep -oPm1 "(?<=<version>)[^<]+" "${srcdir}/extracted/vmware-usbarbitrator/manifest.xml")
    local vmware_ovftool_version=$(grep -oPm1 "(?<=<version>)[^<]+" "${srcdir}/extracted/vmware-ovftool/manifest.xml")
    local vmware_ovftool_build=$(grep -oPm1 "(?<=<buildNumber>)[^<]+" "${srcdir}/extracted/vmware-ovftool/manifest.xml")
    
    #################### Create new empty directories in the package directory. Instead command `install -d -m 755` you can to use `mkdir -p`
    install -d -m 755 \
        "${pkgdir}/etc"/{cups,thnuclnt,modprobe.d,vmware,vmware-installer} \
        "${pkgdir}/usr"/{bin,lib/{modules-load.d,cups/filter},share/licenses/${pkgname}} \
        "${pkgdir}/usr/lib/systemd"/{system,scripts} \
        "${pkgdir}/usr/lib/vmware"/{bin,lib,setup,isoimages,modules} \
        "${pkgdir}/usr/lib/vmware-ovftool"/{certs,env,schemas} \
        "${pkgdir}/usr/lib/vmware-installer/$vmware_installer_version"/{vmis,lib/lib,artwork} \


    #################### Coping files into the package directory
    cp -r "${srcdir}/extracted/vmware-player-app/bin"/* "${pkgdir}/usr/bin"
    cp -r "${srcdir}/extracted/vmware-player-app/lib"/* "${pkgdir}/usr/lib/vmware"
    cp -r "${srcdir}/extracted/vmware-player-app/share"/* "${pkgdir}/usr/share"
    cp -r "${srcdir}/extracted/vmware-player-app/etc/cups"/* "${pkgdir}/etc/cups"
    cp -r "${srcdir}/extracted/vmware-player-app/extras/.thnumod" "${pkgdir}/etc/thnuclnt"
    cp -r "${srcdir}/extracted/vmware-player-app/extras/thnucups" "${pkgdir}/usr/lib/cups/filter"
    cp -r "${srcdir}/extracted/vmware-player-app/doc"/*open_source_licenses.txt "${pkgdir}/usr/share/licenses/${pkgname}"
    cp -T "${srcdir}/extracted/vmware-player/doc/EULA" "${pkgdir}/usr/share/licenses/${pkgname}/VMware Player - EULA.txt"
    cp -r "${srcdir}/extracted/vmware-player-setup/vmware-config" "${pkgdir}/usr/lib/vmware/setup"
    cp -T "${srcdir}/vmware-bootstrap" "${pkgdir}/etc/vmware/bootstrap"
    cp -T "${srcdir}/vmware-config" "${pkgdir}/etc/vmware/config"

    cp -r "${srcdir}/extracted/vmware-vmx"/{,s}bin/* "${pkgdir}/usr/bin"
    cp -r "${srcdir}/extracted/vmware-vmx"/{lib/*,roms} "${pkgdir}/usr/lib/vmware"
    cp -T "${srcdir}/extracted/vmware-vmx/etc/modprobe.d/modprobe-vmware-fuse.conf" "${pkgdir}/etc/modprobe.d/vmware-fuse.conf"
    cp -T "${srcdir}/extracted/vmware-vmx/extra/modules.xml" "${pkgdir}/usr/lib/vmware/modules/modules.xml"
    
    cp -r "${srcdir}/extracted/vmware-ovftool"/* "${pkgdir}/usr/lib/vmware-ovftool"
    mv "${pkgdir}/usr/lib/vmware-ovftool/vmware.eula" "${pkgdir}/usr/share/licenses/${pkgname}/VMware OVF Tool component for Linux - EULA.txt"
    rm "${pkgdir}/usr/lib/vmware-ovftool"/{vmware-eula.rtf,open_source_licenses.txt,manifest.xml}

    cp -r "${srcdir}/extracted/vmware-usbarbitrator/bin" "${pkgdir}/usr/lib/vmware"
    cp -r "${srcdir}/extracted/vmware-network-editor/lib" "${pkgdir}/usr/lib/vmware"
    cp -r "${srcdir}/extracted/vmware-virtual-printer"/VirtualPrinter-*.iso "${pkgdir}/usr/lib/vmware/isoimages"

    cp -r "${srcdir}/extracted/vmware-installer"/{bin,lib/lib,python,sopython,artwork} "${pkgdir}/usr/lib/vmware-installer/$vmware_installer_version"
    install -m 755 "${srcdir}/vmware-configure-initscript.sh" "${pkgdir}/usr/lib/vmware-installer/$vmware_installer_version/bin/configure-initscript.sh"
    cp -r "${srcdir}/extracted/vmware-installer"/{vmis,vmis-launcher,vmware-installer,vmware-installer.py} "${pkgdir}/usr/lib/vmware-installer/$vmware_installer_version"
    cp -T "${srcdir}/extracted/vmware-installer/bootstrap" "${pkgdir}/etc/vmware-installer/bootstrap"

    # Installation a config file of auto load kernel modules: https://www.freedesktop.org/software/systemd/man/modules-load.d.html
    install -m 644 "${srcdir}/vmware-modules-load.conf" "${pkgdir}/usr/lib/modules-load.d/vmware.conf"
    # Installation main sh-script of manager services:
    install -m 755 "${srcdir}/vmware-initd" "${pkgdir}/usr/lib/systemd/scripts/vmware"
    # Installation SystemD files:
    install -m 644 "${srcdir}/vmware-networks-configuration.service" "${pkgdir}/usr/lib/systemd/system/vmware-networks-configuration.service"
    install -m 644 "${srcdir}/vmware-networks.service" "${pkgdir}/usr/lib/systemd/system/vmware-networks.service"
    install -m 644 "${srcdir}/vmware-usbarbitrator.service" "${pkgdir}/usr/lib/systemd/system/vmware-usbarbitrator.service"


    #################### Apply permissions where necessary
    chmod +x \
        "${pkgdir}/usr/bin"/* \
        "${pkgdir}/usr/lib/vmware/bin"/* \
        "${pkgdir}/usr/lib/vmware/setup"/* \
        "${pkgdir}/usr/lib/vmware/lib/libvmware-gksu.so/gksu-run-helper" \
        "${pkgdir}/usr/lib/vmware-ovftool"/{ovftool,ovftool.bin} \
        "${pkgdir}/usr/lib/vmware-installer/$vmware_installer_version"/{vmware-installer,vmis-launcher} \
        "${pkgdir}/usr/lib/cups/filter"/* \
        "${pkgdir}/etc/thnuclnt/.thnumod"

    chmod +s \
        "${pkgdir}/usr/bin/vmware-authd" \
        "${pkgdir}/usr/lib/vmware/bin"/{vmware-vmx,vmware-vmx-debug,vmware-vmx-stats}

    #################### Add symlinks the installer would create
    for link in licenseTool \
        vmplayer \
        vmware-app-control \
        vmware-enter-serial \
        vmware-fuseUI \
        vmware-gksu \
        vmware-modconfig \
        vmware-modconfig-console \
        vmware-mount \
        vmware-netcfg \
        vmware-setup-helper \
        vmware-vmblock-fuse \
        vmware-zenity
    do
        ln -s /usr/lib/vmware/bin/appLoader "${pkgdir}/usr/lib/vmware/bin/$link"
    done

    ln -s /usr/lib/vmware-ovftool/ovftool "${pkgdir}/usr/bin/ovftool"
    ln -s /usr/lib/vmware/bin/appLoader "${pkgdir}/usr/bin/vmrest"
    ln -s /usr/lib/vmware/bin/vmware-fuseUI "${pkgdir}/usr/bin/vmware-fuseUI"
    ln -s /usr/lib/vmware-installer/$vmware_installer_version/vmware-installer "${pkgdir}/usr/bin/vmware-installer"
    ln -s /usr/lib/vmware/bin/vmware-mount "${pkgdir}/usr/bin/vmware-mount"
    ln -s /usr/lib/vmware/bin/vmware-netcfg "${pkgdir}/usr/bin/vmware-netcfg"
    ln -s /usr/lib/vmware/bin/vmware-usbarbitrator "${pkgdir}/usr/bin/vmware-usbarbitrator"
    ln -s /usr/lib/vmware/icu "${pkgdir}/etc/vmware/icu"
    # ln -s /usr/lib/vmware-installer/$vmware_installer_version/vmware-uninstall-downgrade "${pkgdir}/usr/bin/vmware-uninstall"

    # Next a links needed for the GUI vmware-installer wich uses ncurses5-compat-libs.
    # In a Arch distrib uses pakages `ncurses` and `lib32-ncurses` of version 6.x
    ln -s /usr/lib/libncursesw.so.6.1 "${pkgdir}/usr/lib/libncursesw.so.5"
    ln -s /usr/lib/libncursesw.so.6 "${pkgdir}/usr/lib/libtinfo.so.5"


    #################### Replace placeholder "variables" with real paths

    sed -i 's,@@LIBCONF_DIR@@,/usr/lib/vmware/libconf,g' "${pkgdir}/usr/lib/vmware/libconf/etc/gtk-3.0/gdk-pixbuf.loaders"
    sed -i 's,@@BINARY@@,/usr/bin/vmplayer,' "${pkgdir}/usr/share/applications/vmware-player.desktop"

    sed -e "s/@@VERSION@@/$vmware_installer_version/" \
        -e "s,@@VMWARE_INSTALLER@@,/usr/lib/vmware-installer/$vmware_installer_version," \
        -i "${pkgdir}/etc/vmware-installer/bootstrap"

    sed -e "/^player.product.version/s/=.*$/= \"$pkgver\"/" \
        -e "/^product.buildNumber/s/=.*$/= \"$pkgbuild\"/" \
        -i "${pkgdir}/etc/vmware/config"
    

    #################### Patch up the VMware kernel sources and configure DKMS


    #################### Create a database which contains the list of guest tools (necessary to avoid that vmware try to download them)
    local database_filename="${pkgdir}/etc/vmware-installer/database"
    echo -n "" > "$database_filename"

    sqlite3 "$database_filename" "
        CREATE TABLE settings(key VARCHAR PRIMARY KEY, 
                              value VARCHAR NOT NULL,
                              component_name VARCHAR NOT NULL);
        INSERT INTO settings(key,value,component_name) VALUES('db.schemaVersion','2','vmware-installer');
        INSERT INTO settings(key,value,component_name) VALUES('libconf','','vmware-installer');
        INSERT INTO settings(key,value,component_name) VALUES('prefix.asked','','vmware-installer');
        INSERT INTO settings(key,value,component_name) VALUES('prefix','/usr','vmware-installer');
        INSERT INTO settings(key,value,component_name) VALUES('initdir.asked','','vmware-installer');
        INSERT INTO settings(key,value,component_name) VALUES('initdir','/etc','vmware-installer');
        INSERT INTO settings(key,value,component_name) VALUES('initscriptdir.asked','','vmware-installer');
        INSERT INTO settings(key,value,component_name) VALUES('initscriptdir','/usr/lib/systemd/scripts','vmware-installer');
        INSERT INTO settings(key,value,component_name) VALUES('installShortcuts.asked','','vmware-installer');
        INSERT INTO settings(key,value,component_name) VALUES('installShortcuts','yes','vmware-installer');
        INSERT INTO settings(key,value,component_name) VALUES('softwareUpdateEnabled.asked','Yes','vmware-player-app');
        INSERT INTO settings(key,value,component_name) VALUES('softwareUpdateEnabled','no','vmware-player-app');
        INSERT INTO settings(key,value,component_name) VALUES('dataCollectionEnabled.asked','Yes','vmware-player-app');
        INSERT INTO settings(key,value,component_name) VALUES('dataCollectionEnabled','no','vmware-player-app');
        INSERT INTO settings(key,value,component_name) VALUES('serialNumber.asked','Yes','vmware-player');
        INSERT INTO settings(key,value,component_name) VALUES('vmware-player.eula','yes','vmware-player');
        INSERT INTO settings(key,value,component_name) VALUES('serialNumber','','vmware-player');
        INSERT INTO settings(key,value,component_name) VALUES('ClosePrograms.asked','Yes','vmware-vmx');
        INSERT INTO settings(key,value,component_name) VALUES('ClosePrograms','Press enter to continue.','vmware-vmx');
        INSERT INTO settings(key,value,component_name) VALUES('${vmware_installer_version}.vmisloc','/usr/lib/vmware-installer/${vmware_installer_version}','vmware-installer');
        INSERT INTO settings(key,value,component_name) VALUES('${vmware_installer_version}.pyloc','/usr/lib/vmware-installer/${vmware_installer_version}/python','vmware-installer');
        INSERT INTO settings(key,value,component_name) VALUES('${vmware_installer_version}.pyver','27','vmware-installer');
        INSERT INTO settings(key,value,component_name) VALUES('currentVersion','${vmware_installer_version}','vmware-installer');
    "

    sqlite3 "$database_filename" "
        CREATE TABLE components(id INTEGER PRIMARY KEY,
                                name VARCHAR NOT NULL,
                                version VARCHAR NOT NULL,
                                buildNumber INTEGER NOT NULL,
                                component_core_id INTEGER NOT NULL,
                                longName VARCHAR NOT NULL,
                                description VARCHAR,
                                type INTEGER NOT NULL);
        INSERT INTO components VALUES(1,'vmware-installer','${vmware_installer_version}',${pkgbuild},-1,'VMware Installer','VMware Installer',1);
        INSERT INTO components VALUES(2,'vmware-player-setup','${pkgver}',${pkgbuild},1,'VMware Player Setup','VMware Player Setup',1);
        INSERT INTO components VALUES(3,'vmware-vmx','${pkgver}',${pkgbuild},1,'VMware VMX','VMware Linux VMX',1);
        INSERT INTO components VALUES(4,'vmware-virtual-printer','1.0',${pkgbuild},1,'VMware Virtual Printer','Virtual Printer Component',1);
        INSERT INTO components VALUES(5,'vmware-network-editor','${pkgver}',${pkgbuild},1,'VMware Network Editor','Network Editor Component for Linux',1);
        INSERT INTO components VALUES(6,'vmware-usbarbitrator','${vmware_usbarbitrator_version}',${pkgbuild},1,'VMware USB Arbitrator','USB Arbitrator Component for Linux',1);
        INSERT INTO components VALUES(7,'vmware-player-app','${pkgver}',${pkgbuild},1,'VMware Player Application','VMware Player Application for Linux',1);
        INSERT INTO components VALUES(8,'vmware-ovftool','${vmware_ovftool_version}',${vmware_ovftool_build},1,'VMware OVF Tool component for Linux','VMware OVF Tool is a command-line utility that allows you to import and export OVF packages to and from many VMware products.',1);
        INSERT INTO components VALUES(9,'vmware-player','${pkgver}',${pkgbuild},1,'VMware Player','VMware Player for Linux',0);
    "

    # for isoimage in linux linuxPreGlibc25 netware solaris windows winPre2k winPreVista
    # do
    #     local version=$(cat "$srcdir/extracted/vmware-tools-$isoimage/manifest.xml" | grep -oPm1 "(?<=<version>)[^<]+")
    #     sqlite3 "$database_filename" "INSERT INTO components(name,version,buildNumber,component_core_id,longName,description,type) VALUES(\"vmware-tools-$isoimage\",\"$version\",\"${_pkgver#*_}\",1,\"$isoimage\",\"$isoimage\",1);"
    # done

    sqlite3 "$database_filename" "
        CREATE TABLE component_conflicts(component_id INTEGER NOT NULL, conflict VARCHAR NOT NULL);
        INSERT INTO component_conflicts VALUES(9,'vmware-workstation>=0.0.1');
    "

    sqlite3 "$database_filename" "
        CREATE TABLE component_dependencies(component_id INTEGER NOT NULL, dependency VARCHAR NOT NULL);
        INSERT INTO component_dependencies VALUES(2,'vmware-installer=${vmware_installer_version}');
        INSERT INTO component_dependencies VALUES(3,'vmware-installer=${vmware_installer_version}');
        INSERT INTO component_dependencies VALUES(3,'vmware-player-setup=${pkgver}');
        INSERT INTO component_dependencies VALUES(4,'vmware-installer=${vmware_installer_version}');
        INSERT INTO component_dependencies VALUES(5,'vmware-installer=${vmware_installer_version}');
        INSERT INTO component_dependencies VALUES(6,'vmware-installer=${vmware_installer_version}');
        INSERT INTO component_dependencies VALUES(7,'vmware-installer=${vmware_installer_version}');
        INSERT INTO component_dependencies VALUES(7,'vmware-player-setup=${pkgver}');
        INSERT INTO component_dependencies VALUES(7,'vmware-usbarbitrator>=${vmware_usbarbitrator_version}');
        INSERT INTO component_dependencies VALUES(7,'vmware-network-editor=${pkgver}');
        INSERT INTO component_dependencies VALUES(7,'vmware-vmx=${pkgver}');
        INSERT INTO component_dependencies VALUES(7,'vmware-virtual-printer=1.0');
        INSERT INTO component_dependencies VALUES(7,'Opt:vmware-tools-freebsd>=10.3.2');
        INSERT INTO component_dependencies VALUES(7,'Opt:vmware-tools-linuxPreGlibc25>=10.3.2');
        INSERT INTO component_dependencies VALUES(7,'Opt:vmware-tools-netware>=10.3.2');
        INSERT INTO component_dependencies VALUES(7,'Opt:vmware-tools-solaris>=10.3.2');
        INSERT INTO component_dependencies VALUES(7,'Opt:vmware-tools-winPre2k>=10.3.2');
        INSERT INTO component_dependencies VALUES(7,'Opt:vmware-tools-winPreVista>=10.3.2');
        INSERT INTO component_dependencies VALUES(7,'Opt:vmware-tools-windows>=10.3.2');
        INSERT INTO component_dependencies VALUES(8,'vmware-installer=${vmware_installer_version}');
        INSERT INTO component_dependencies VALUES(9,'vmware-installer=${vmware_installer_version}');
        INSERT INTO component_dependencies VALUES(9,'vmware-player-app=${pkgver}');
        INSERT INTO component_dependencies VALUES(9,'vmware-ovftool>=${vmware_ovftool_version}');
        INSERT INTO component_dependencies VALUES(9,'vmware-vmx=${pkgver}');
    "

    sqlite3 "$database_filename" "
        CREATE TABLE component_reverse_dependencies(component_id INTEGER NOT NULL,
                                                    name VARCHAR NOT NULL);
    "

    sqlite3 "$database_filename" "
        CREATE TABLE files(id INTEGER PRIMARY KEY,
                         path VARCHAR NOT NULL UNIQUE,
                         mtime INTEGER NOT NULL,
                         type INTEGER NOT NULL,
                         component_id INTEGER);
    "
    
}
