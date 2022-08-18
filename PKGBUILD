# Maintainer: Essem <smswessem@gmail.com>

# Based on both the official Arch nvidia packages and the nvidia-merged AUR packages

pkgbase='nvidia-merged-unlocked'
pkgname=('lib32-nvidia-merged-unlocked-utils' 'lib32-opencl-nvidia-merged-unlocked' 'nvidia-merged-unlocked-dkms' 'nvidia-merged-unlocked-settings' 'nvidia-merged-unlocked-utils' 'opencl-nvidia-merged-unlocked')
pkgver=510.85.02
pkgrel=1
arch=('x86_64')
makedepends=('git' 'rust')
url='https://github.com/VGPU-Community-Drivers/vGPU-Unlock-patcher'
license=('custom')
options=('!strip')
groups=('nvidia-merged-unlocked')
conflicts=('nvidia-merged')

_vgpupkgver=510.85.03
_basepkg="NVIDIA-Linux-${CARCH}-${pkgver}"
_vgpupkg="NVIDIA-Linux-${CARCH}-${_vgpupkgver}-vgpu-kvm"
_mergedpkg="${_basepkg}-merged-vgpu-kvm"
_patchedpkg="${_mergedpkg}-patched"

source=(
    "nvidia-vgpu.conf"
    "nvidia-drm-outputclass.conf"
    "nvidia-utils.sysusers"
    "nvidia.rules"
    #"1070.patch"
    "nvidia-510.73.05-vgpu-5.18.patch"
    "https://us.download.nvidia.com/XFree86/Linux-x86_64/${pkgver}/${_basepkg}.run"
    "https://github.com/VGPU-Community-Drivers/NV-VGPU-Driver/releases/download/1.0.2/${_vgpupkg}.run"
    'git+https://github.com/VGPU-Community-Drivers/vGPU-Unlock-patcher')
sha256sums=(
            '5ea0d9edfcf282cea9b204291716a9a4d6d522ba3a6bc28d78edf505b6dc7949'
            'be99ff3def641bb900c2486cce96530394c5dc60548fc4642f19d3a4c784134d'
            'b9c775b87829698cee92cd8b0004bd3d34b8b5cb36cef4d96199539fa72f8d15'
            'ae08ef9a33d3bb4218c0165cecc9b8a7cab03bfd749cdbd0d245cfc274852c4b'
            #'4813d9dc351aea6a20f2c8911c40c0bf0360e2f9a484934f965d40af40cf38c1'
            '79b09682f9c3cfa32d219a14b3c956d8f05340addd5a43e7063c3d96175a56f4'
            '372427e633f32cff6dd76020e8ed471ef825d38878bd9655308b6efea1051090'
            '773d3a215cedf5349eff00a9b052bc961f51ce971933f154f2d2997ed68aa59a'
            'SKIP')

create_links() {
    # create soname links
    find "$pkgdir" -type f -name '*.so*' ! -path '*xorg/*' -print0 | while read -d $'\0' _lib; do
        _soname=$(dirname "${_lib}")/$(readelf -d "${_lib}" | grep -Po 'SONAME.*: \[\K[^]]*' || true)
        _base=$(echo ${_soname} | sed -r 's/(.*)\.so.*/\1.so/')
        [[ -e "${_soname}" ]] || ln -s $(basename "${_lib}") "${_soname}"
        [[ -e "${_base}" ]] || ln -s $(basename "${_soname}") "${_base}"
    done
}

prepare() {
    cd "${srcdir}/vGPU-Unlock-patcher"
    git submodule update --init

    #patch -p1 < "${srcdir}/1070.patch"
    sed \
      -e 's|GNRL="N|GNRL="../N|' \
      -e 's|VGPU="N|VGPU="../N|' \
      -e 's|GRID="N|GRID="../N|' \
      -i patch.sh

    ./patch.sh --no-opt-vgpu general-merge

    cd "${srcdir}/${_patchedpkg}"

    sed \
      -e 's|__UTILS_PATH__|/usr/bin|' \
      -e 's|Icon=.*|Icon=nvidia-settings|' \
      -i nvidia-settings.desktop

    bsdtar -xf nvidia-persistenced-init.tar.bz2

    sed \
      -e "s/__VERSION_STRING/${pkgver}/" \
      -e 's/__JOBS/`nproc`/' \
      -e 's/__DKMS_MODULES//' \
      -e '$iBUILT_MODULE_NAME[0]="nvidia"\
DEST_MODULE_LOCATION[0]="/kernel/drivers/video"\
BUILT_MODULE_NAME[1]="nvidia-uvm"\
DEST_MODULE_LOCATION[1]="/kernel/drivers/video"\
BUILT_MODULE_NAME[2]="nvidia-modeset"\
DEST_MODULE_LOCATION[2]="/kernel/drivers/video"\
BUILT_MODULE_NAME[3]="nvidia-drm"\
DEST_MODULE_LOCATION[3]="/kernel/drivers/video"\
BUILT_MODULE_NAME[4]="nvidia-vgpu-vfio"\
DEST_MODULE_LOCATION[4]="/kernel/drivers/video" \
BUILT_MODULE_NAME[5]="nvidia-peermem"\
DEST_MODULE_LOCATION[5]="/kernel/drivers/video"' \
      -e 's/NV_EXCLUDE_BUILD_MODULES/IGNORE_PREEMPT_RT_PRESENCE=1 NV_EXCLUDE_BUILD_MODULES/' \
      -i kernel/dkms.conf

    pushd "${srcdir}/${_patchedpkg}"
    patch -p1 < "${srcdir}/nvidia-510.73.05-vgpu-5.18.patch"
    popd

    # Gift for linux-rt guys
    sed -i 's/NV_EXCLUDE_BUILD_MODULES/IGNORE_PREEMPT_RT_PRESENCE=1 NV_EXCLUDE_BUILD_MODULES/' kernel/dkms.conf
}

package_opencl-nvidia-merged-unlocked() {
    pkgdesc="OpenCL implemention for NVIDIA"
    depends=('zlib')
    optdepends=('opencl-headers: headers necessary for OpenCL development')
    provides=('opencl-driver' 'opencl-nvidia')
    conflicts=('opencl-nvidia' 'opencl-nvidia-merged')

    cd "${srcdir}"

    # OpenCL
    install -D -m644 "${_patchedpkg}/nvidia.icd" "${pkgdir}/etc/OpenCL/vendors/nvidia.icd"
    install -D -m755 "${_patchedpkg}/libnvidia-compiler-next.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-compiler-next.so.${pkgver}"
    install -D -m755 "${_patchedpkg}/libnvidia-compiler.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-compiler.so.${pkgver}"
    install -D -m755 "${_patchedpkg}/libnvidia-opencl.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-opencl.so.${pkgver}"

    create_links

    install -D -m644 "${_patchedpkg}/LICENSE" "${pkgdir}/usr/share/licenses/$pkgname/LICENSE"
}

package_nvidia-merged-unlocked-dkms() {
    pkgdesc="NVIDIA drivers - module sources; patched for vGPU support w/ unlock & host DRM output"
    depends=('dkms' "nvidia-merged-unlocked-utils=${pkgver}" 'libglvnd')
    provides=('NVIDIA-MODULE' 'nvidia-dkms')
    conflicts=('nvidia-dkms' 'nvidia-merged-dkms')

    cd "${srcdir}"

    install -d -m755 "${pkgdir}/usr/src"
    cp -dr --no-preserve='ownership' "${_patchedpkg}/kernel" "${pkgdir}/usr/src/nvidia-${pkgver}"

    echo "blacklist nouveau" | install -D -m644 /dev/stdin "${pkgdir}/usr/lib/modprobe.d/${pkgname}.conf"
    echo "nvidia-uvm"        | install -D -m644 /dev/stdin "${pkgdir}/usr/lib/modules-load.d/${pkgname}.conf"

    install -D -m644 "${_patchedpkg}/LICENSE" "${pkgdir}/usr/share/licenses/$pkgname/LICENSE"
}

package_nvidia-merged-unlocked-settings() {
    pkgdesc='Tool for configuring the NVIDIA graphics driver'
    depends=('jansson' 'gtk3' 'libxv' 'libvdpau' "nvidia-merged-unlocked-utils=${pkgver}")
    provides=('nvidia-settings')
    conflicts=('nvidia-settings' 'nvidia-merged-settings')

    cd "${srcdir}"

    install -D -m755 "${_patchedpkg}/nvidia-settings" "${pkgdir}/usr/bin/nvidia-settings"
    install -D -m644 "${_patchedpkg}/libnvidia-gtk3.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-gtk3.so.${pkgver}"
    install -D -m644 "${_patchedpkg}/nvidia-settings.1.gz" "${pkgdir}/usr/share/man/man1/nvidia-settings.1.gz"
    install -D -m644 "${_patchedpkg}/nvidia-settings.png" "${pkgdir}/usr/share/pixmaps/nvidia-settings.png"
    install -D -m644 "${_patchedpkg}/nvidia-settings.desktop" "${pkgdir}/usr/share/applications/nvidia-settings.desktop"

    create_links

    install -D -m644 "${_patchedpkg}/LICENSE" "${pkgdir}/usr/share/licenses/$pkgname/LICENSE"
}

package_nvidia-merged-unlocked-utils() {
    pkgdesc="NVIDIA drivers utilities; patched for vGPU support w/ unlock & host DRM output"
    depends=('xorg-server' 'libglvnd' 'egl-wayland')
    optdepends=("nvidia-merged-unlocked-settings=${pkgver}: configuration tool"
                'xorg-server-devel: nvidia-xconfig'
                "opencl-nvidia-merged-unlocked=${pkgver}: OpenCL support"
                'mdevctl: mediated device contfiguration tool'
                'libvirt: virtualization engine control interface')
    conflicts=('nvidia-libgl' 'nvidia-utils' 'nvidia-merged-unlocked-utils')
    provides=('vulkan-driver' 'opengl-driver' 'nvidia-libgl' 'nvidia-utils' 'vgpu_unlock')
    replaces=('vgpu_unlock')
    cd "${srcdir}/${_patchedpkg}"

    # X driver
    install -Dm755 nvidia_drv.so "${pkgdir}/usr/lib/xorg/modules/drivers/nvidia_drv.so"

    # Wayland/GBM
    install -Dm755 libnvidia-egl-gbm.so.1* -t "${pkgdir}/usr/lib/"
    install -Dm644 15_nvidia_gbm.json "${pkgdir}/usr/share/egl/egl_external_platform.d/15_nvidia_gbm.json"
    mkdir -p "${pkgdir}/usr/lib/gbm"
    ln -sr "${pkgdir}/usr/lib/libnvidia-allocator.so.${pkgver}" "${pkgdir}/usr/lib/gbm/nvidia-drm_gbm.so"

    # firmware
    install -Dm644 firmware/gsp.bin "${pkgdir}/usr/lib/firmware/nvidia/${pkgver}/gsp.bin"

    # GLX extension module for X
    install -Dm755 "libglxserver_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so.${pkgver}"
    # Ensure that X finds glx
    ln -s "libglxserver_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so.1"
    ln -s "libglxserver_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/nvidia/xorg/libglxserver_nvidia.so"

    install -Dm755 "libGLX_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libGLX_nvidia.so.${pkgver}"

    # OpenGL libraries
    install -Dm755 "libEGL_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libEGL_nvidia.so.${pkgver}"
    install -Dm755 "libGLESv1_CM_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libGLESv1_CM_nvidia.so.${pkgver}"
    install -Dm755 "libGLESv2_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/libGLESv2_nvidia.so.${pkgver}"
    install -Dm644 "10_nvidia.json" "${pkgdir}/usr/share/glvnd/egl_vendor.d/10_nvidia.json"

    # OpenGL core library
    install -Dm755 "libnvidia-glcore.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-glcore.so.${pkgver}"
    install -Dm755 "libnvidia-eglcore.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-eglcore.so.${pkgver}"
    install -Dm755 "libnvidia-glsi.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-glsi.so.${pkgver}"

    # misc
    install -Dm755 "libnvidia-fbc.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-fbc.so.${pkgver}"
    install -Dm755 "libnvidia-encode.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-encode.so.${pkgver}"
    install -Dm755 "libnvidia-cfg.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-cfg.so.${pkgver}"
    install -Dm755 "libnvidia-ml.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-ml.so.${pkgver}"
    install -Dm755 "libnvidia-glvkspirv.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-glvkspirv.so.${pkgver}"
    install -Dm755 "libnvidia-allocator.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-allocator.so.${pkgver}"
    install -Dm755 "libnvidia-vulkan-producer.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-vulkan-producer.so.${pkgver}"
    # Sigh libnvidia-vulkan-producer.so has no SONAME set so create_links doesn't catch it. NVIDIA please fix!
    ln -s "libnvidia-vulkan-producer.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-vulkan-producer.so.1"
    ln -s "libnvidia-vulkan-producer.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-vulkan-producer.so"

    # Vulkan ICD
    install -Dm644 "nvidia_icd.json" "${pkgdir}/usr/share/vulkan/icd.d/nvidia_icd.json"
    install -Dm644 "nvidia_layers.json" "${pkgdir}/usr/share/vulkan/implicit_layer.d/nvidia_layers.json"

    # VDPAU
    install -Dm755 "libvdpau_nvidia.so.${pkgver}" "${pkgdir}/usr/lib/vdpau/libvdpau_nvidia.so.${pkgver}"

    # nvidia-tls library
    install -Dm755 "libnvidia-tls.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-tls.so.${pkgver}"

    # CUDA
    install -Dm755 "libcuda.so.${pkgver}" "${pkgdir}/usr/lib/libcuda.so.${pkgver}"
    install -Dm755 "libnvcuvid.so.${pkgver}" "${pkgdir}/usr/lib/libnvcuvid.so.${pkgver}"

    # PTX JIT Compiler (Parallel Thread Execution (PTX) is a pseudo-assembly language for CUDA)
    install -Dm755 "libnvidia-ptxjitcompiler.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-ptxjitcompiler.so.${pkgver}"

    # raytracing
    install -Dm755 "libnvoptix.so.${pkgver}" "${pkgdir}/usr/lib/libnvoptix.so.${pkgver}"
    install -Dm755 "libnvidia-rtcore.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-rtcore.so.${pkgver}"

    # NGX
    install -Dm755 nvidia-ngx-updater "${pkgdir}/usr/bin/nvidia-ngx-updater"
    install -Dm755 "libnvidia-ngx.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-ngx.so.${pkgver}"
    install -Dm755 _nvngx.dll "${pkgdir}/usr/lib/nvidia/wine/_nvngx.dll"
    install -Dm755 nvngx.dll "${pkgdir}/usr/lib/nvidia/wine/nvngx.dll"

    # Optical flow
    install -Dm755 "libnvidia-opticalflow.so.${pkgver}" "${pkgdir}/usr/lib/libnvidia-opticalflow.so.${pkgver}"

    # vGPU
    install -Dm755 "libnvidia-vgpu.so.${_vgpupkgver}" "${pkgdir}/usr/lib/libnvidia-vgpu.so.${pkgver}"
    install -Dm755 "libnvidia-vgxcfg.so.${_vgpupkgver}" "${pkgdir}/usr/lib/libnvidia-vgxcfg.so.${pkgver}"

    # DEBUG
    install -Dm755 nvidia-debugdump "${pkgdir}/usr/bin/nvidia-debugdump"

    # nvidia-xconfig
    install -Dm755 nvidia-xconfig "${pkgdir}/usr/bin/nvidia-xconfig"
    install -Dm644 nvidia-xconfig.1.gz "${pkgdir}/usr/share/man/man1/nvidia-xconfig.1.gz"

    # nvidia-bug-report
    install -Dm755 nvidia-bug-report.sh "${pkgdir}/usr/bin/nvidia-bug-report.sh"

    # nvidia-smi
    install -Dm755 nvidia-smi "${pkgdir}/usr/bin/nvidia-smi"
    install -Dm644 nvidia-smi.1.gz "${pkgdir}/usr/share/man/man1/nvidia-smi.1.gz"
    #install -D -m755 "${srcdir}/nvidia-smi" "${pkgdir}/usr/bin/nvidia-smi"

    # nvidia-vgpu
    install -D -m755 nvidia-vgpud "${pkgdir}/usr/bin/nvidia-vgpud"
    install -D -m755 nvidia-vgpu-mgr "${pkgdir}/usr/bin/nvidia-vgpu-mgr"
    install -D -m644 vgpuConfig.xml "${pkgdir}/usr/share/nvidia/vgpu/vgpuConfig.xml"
    install -D -m644 init-scripts/systemd/nvidia-vgpud.service "${pkgdir}/usr/lib/systemd/system/nvidia-vgpud.service"
    install -D -m644 init-scripts/systemd/nvidia-vgpu-mgr.service "${pkgdir}/usr/lib/systemd/system/nvidia-vgpu-mgr.service"
    install -D -m644 "${srcdir}/nvidia-vgpu.conf" "${pkgdir}/usr/lib/systemd/system/libvirtd.service.d/20-nvidia-vgpu.conf"

    # nvidia-cuda-mps
    install -Dm755 nvidia-cuda-mps-server "${pkgdir}/usr/bin/nvidia-cuda-mps-server"
    install -Dm755 nvidia-cuda-mps-control "${pkgdir}/usr/bin/nvidia-cuda-mps-control"
    install -Dm644 nvidia-cuda-mps-control.1.gz "${pkgdir}/usr/share/man/man1/nvidia-cuda-mps-control.1.gz"

    # nvidia-modprobe
    # This should be removed if nvidia fixed their uvm module!
    install -Dm4755 nvidia-modprobe "${pkgdir}/usr/bin/nvidia-modprobe"
    install -Dm644 nvidia-modprobe.1.gz "${pkgdir}/usr/share/man/man1/nvidia-modprobe.1.gz"

    # nvidia-persistenced
    install -Dm755 nvidia-persistenced "${pkgdir}/usr/bin/nvidia-persistenced"
    install -Dm644 nvidia-persistenced.1.gz "${pkgdir}/usr/share/man/man1/nvidia-persistenced.1.gz"
    install -Dm644 nvidia-persistenced-init/systemd/nvidia-persistenced.service.template "${pkgdir}/usr/lib/systemd/system/nvidia-persistenced.service"
    sed -i 's/__USER__/nvidia-persistenced/' "${pkgdir}/usr/lib/systemd/system/nvidia-persistenced.service"

    # application profiles
    install -Dm644 nvidia-application-profiles-${pkgver}-rc "${pkgdir}/usr/share/nvidia/nvidia-application-profiles-${pkgver}-rc"
    install -Dm644 nvidia-application-profiles-${pkgver}-key-documentation "${pkgdir}/usr/share/nvidia/nvidia-application-profiles-${pkgver}-key-documentation"

    install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
    install -Dm644 README.txt "${pkgdir}/usr/share/doc/nvidia/README"
    install -Dm644 NVIDIA_Changelog "${pkgdir}/usr/share/doc/nvidia/NVIDIA_Changelog"
    cp -r html "${pkgdir}/usr/share/doc/nvidia/"
    ln -s nvidia "${pkgdir}/usr/share/doc/nvidia-utils"

    # distro specific files must be installed in /usr/share/X11/xorg.conf.d
    install -Dm644 "${srcdir}/nvidia-drm-outputclass.conf" "${pkgdir}/usr/share/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf"

    install -Dm644 "${srcdir}/nvidia-utils.sysusers" "${pkgdir}/usr/lib/sysusers.d/$pkgname.conf"

    install -Dm644 "${srcdir}/nvidia.rules" "$pkgdir"/usr/lib/udev/rules.d/60-nvidia.rules

    create_links
}

package_lib32-nvidia-merged-unlocked-utils() {
    pkgdesc="NVIDIA drivers utilities; patched for vGPU support w/ unlock & host DRM output (32-bit)"
    depends=('lib32-zlib' 'lib32-gcc-libs' 'lib32-libglvnd' "nvidia-merged-unlocked-utils=${pkgver}")
    optdepends=("lib32-opencl-nvidia=${pkgver}")
    provides=('lib32-vulkan-driver' 'lib32-opengl-driver' 'lib32-nvidia-libgl' 'lib32-nvidia-utils')
    replaces=('lib32-nvidia-libgl')
    conflicts=('lib32-nvidia-utils' 'lib32-nvidia-merged-utils')

    cd "${srcdir}/${_patchedpkg}/32"

    install -D -m755 "libGLX_nvidia.so.${pkgver}" "${pkgdir}/usr/lib32/libGLX_nvidia.so.${pkgver}"

    # OpenGL libraries
    install -D -m755 "libEGL_nvidia.so.${pkgver}" "${pkgdir}/usr/lib32/libEGL_nvidia.so.${pkgver}"
    install -D -m755 "libGLESv1_CM_nvidia.so.${pkgver}" "${pkgdir}/usr/lib32/libGLESv1_CM_nvidia.so.${pkgver}"
    install -D -m755 "libGLESv2_nvidia.so.${pkgver}" "${pkgdir}/usr/lib32/libGLESv2_nvidia.so.${pkgver}"

    # OpenGL core library
    install -D -m755 "libnvidia-glcore.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-glcore.so.${pkgver}"
    install -D -m755 "libnvidia-eglcore.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-eglcore.so.${pkgver}"
    install -D -m755 "libnvidia-glsi.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-glsi.so.${pkgver}"

    # misc
    install -D -m755 "libnvidia-fbc.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-fbc.so.${pkgver}"
    install -D -m755 "libnvidia-encode.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-encode.so.${pkgver}"
    install -D -m755 "libnvidia-ml.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-ml.so.${pkgver}"
    install -D -m755 "libnvidia-glvkspirv.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-glvkspirv.so.${pkgver}"
    install -D -m755 "libnvidia-allocator.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-allocator.so.${pkgver}"

    # VDPAU
    install -D -m755 "libvdpau_nvidia.so.${pkgver}" "${pkgdir}/usr/lib32/vdpau/libvdpau_nvidia.so.${pkgver}"

    # nvidia-tls library
    install -D -m755 "libnvidia-tls.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-tls.so.${pkgver}"

    # CUDA
    install -D -m755 "libcuda.so.${pkgver}" "${pkgdir}/usr/lib32/libcuda.so.${pkgver}"
    install -D -m755 "libnvcuvid.so.${pkgver}" "${pkgdir}/usr/lib32/libnvcuvid.so.${pkgver}"

    # PTX JIT Compiler (Parallel Thread Execution (PTX) is a pseudo-assembly language for CUDA)
    install -D -m755 "libnvidia-ptxjitcompiler.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-ptxjitcompiler.so.${pkgver}"

    # Optical flow
    install -D -m755 "libnvidia-opticalflow.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-opticalflow.so.${pkgver}"

    create_links

    install -D -m644 ../LICENSE "${pkgdir}/usr/share/licenses/$pkgname/LICENSE"
}

package_lib32-opencl-nvidia-merged-unlocked() {
    pkgdesc="OpenCL implemention for NVIDIA (32-bit)"
    depends=('lib32-zlib' 'lib32-gcc-libs')
    optdepends=('opencl-headers: headers necessary for OpenCL development')
    provides=('lib32-opencl-driver' 'lib32-opencl-nvidia')
    conflicts=('lib32-opencl-nvidia' 'lib32-opencl-nvidia-merged')

    cd "${srcdir}/${_patchedpkg}/32"

    # OpenCL
    install -D -m755 "libnvidia-compiler.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-compiler.so.${pkgver}"
    install -D -m755 "libnvidia-opencl.so.${pkgver}" "${pkgdir}/usr/lib32/libnvidia-opencl.so.${pkgver}"

    create_links

    install -D -m644 ../LICENSE "${pkgdir}/usr/share/licenses/$pkgname/LICENSE"
}
