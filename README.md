fuseloop
========

FUSE를 사용하는 작은 루프백 마운트 유틸리티입니다.

mke2fs가 전체 파일 미만에서 실행될 수 있도록 특정 offset/size 액세스를 제공하여 가상 디스크 이미지를 생성할 수 있도록 하기 위한 것입니다.

가상 디스크 이미지가 분할되고 파일 시스템이 생성되면 'mountlo'를 사용하여 이를 마운트할 수 있습니다.

Example:

    # 새 디스크 이미지 파일을 만듭니다. (300M in size)
    truncate -s 300M mydisk.img
    
    # fdisk를 사용하여 100M 파티션 생성
    # 헤드/섹터/섹터 크기 실린더에 64/32/512를 사용하는 경우=$DISK_SIZE_IN_M
    fdisk -c -u -C 300 -H 64 -S 32 -b 512 mydisk.img << EOF
    n
    p
    1
    
    +100M
    w
    EOF
    sectorsize=512
    
    # fdisk를 사용하여 fuseloop 와 함께 사용할 올바른 오프셋 및 크기 값을 지정.
    fdisk -c -u -l -C 300 -H 64 -S 32 -b 512 mydisk.img | grep mydisk.img | grep Linux | while read line
    do
      set -- $line
      offs=$(($2*$sectorsize))
      size=$((($3-$2)*$sectorsize))
      
      # 임의 이름으로  파티션 device 지정
      part=part.$$
      touch "$part"
      
      # Use fuseloop to expose the partition as an individual file/device
      fuseloop -O "$offs" -S "$size" mydisk.img "$part"
      
      # Create the file system
      mke2fs -F "$part"
      
      # Unmount the partition
      fusermount -u "$part"
      
      # Clean up the partition file we mounted on
      rm -f "$part"
    done
    
    # 원하는 경우 SYSLinux MBR을 설치하여 부팅 가능한 이미지를 얻을 수 있습니다(파티션 1도 '활성'으로 표시하는 것을 기억하세요!)
    dd if=/usr/lib/syslinux/mbr.bin of=mydisk.img conv=notrunc
    
    # 파티션에 파일 시스템이 배치되어 있으면 mountlo를 사용하여 이를 마운트할 수 있습니다.
    mountlo -p1 -w mydisk.img /mnt
  
  
