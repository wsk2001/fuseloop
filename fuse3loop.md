## fuse3loop 컴파일 및 테스트
``` bash
# 컴파일

gcc -Wall fuse3loop.c -o fuse3loop -lfuse3
# 또는
gcc -o fuse3loop fuse3loop.c `pkg-config fuse3 --cflags --libs`

# image_path=/home/tssuser/tss3/disk.img
# loop_path=/home/tssuser/tss3/.tss3loop_db
# mount_point=/home/tssuser/tss3/mnt


# 테스트용 Image 파일 생성
# fallocate -l 500M $image_path
fallocate -l 500M disk.img

# loop device file 생성
# touch $loop_path
touch .tss3loop_db

# loop 파일과 image 연결
# ./fuse3loop -o allow_other $image_path $loop_path &
./fuse3loop -o allow_other /home/tssuser/tss3/disk.img /home/tssuser/tss3/.tss3loop_db &

# 파일 시스템 생성
# mkfs.ext4 -F $loop_path
mkfs.ext4 -F /home/tssuser/tss3/.tss3loop_db

# mount
# mount $loop_path $mount_point
mount /home/tssuser/tss3/.tss3loop_db /home/tssuser/tss3/mnt

# test
# cd $mount_point
cd /home/tssuser/tss3/mnt
ls -al
# -> 출력
/home/tssuser/tss3/mnt# ls -al
total 24
drwxr-xr-x 3 root    root     4096  9월 25 10:16 .
drwxrwxr-x 3 tssuser tssuser  4096  9월 25 10:10 ..
drwx------ 2 root    root    16384  9월 25 10:16 lost+found

# 테스트 파일 생성
echo aaaaaaaaaaaaaaaaaaaaaaaaaaaaa > aa.txt
echo bbbbbbbbbbbbbbbbbbbbbbbbbbbbbb > bb.txt

cd ..

# unmount
# umount $mount_point
umount /home/tssuser/tss3/mnt

# fusermount -u $loop_path
fusermount -u /home/tssuser/tss3/.tss3loop_db


# 재실행 테스트
./fuse3loop -o allow_other /home/tssuser/tss3/disk.img /home/tssuser/tss3/.tss3loop_db &
mount /home/tssuser/tss3/.tss3loop_db /home/tssuser/tss3/mnt

cd /home/tssuser/tss3/mnt
ls -al

cd ..
umount /home/tssuser/tss3/mnt
fusermount -u /home/tssuser/tss3/.tss3loop_db
```
