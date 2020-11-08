# 구동중인 AWS EC2 디스크 용량 늘이기


## 들어가며

서비스 도중에 EC2 disk 사이즈가 가득 찼다면, 아래와 같이 처리 가능하다. 먼저
AWS Console 또는 AWS CLI를 이용하여 Volume 사이즈를 늘이고, Volume이 연결된 EC2 shell에 연결하여 Partition 확장을 해야한다. 아래와 같이 진행하면된다.

## EC2 Volume Size 수정
- AWS Consule에서 아래 경로로 이동
    - Services > EC2 > Volumes 
- 수정하려는 볼륨을 선택 > Actions - Modify Volume(볼륨수정) 선택 
- 수정을 원하는 Size(크기) 입력

위와 같이 진행하고 나면, 상태가 in-use - optimizing으로 변경된다. 이 상태가 is-use completed가 되고나면 파티션 확장 작업을 진행하자.

---

## Partition 확장
아래와 같이 lsblk 명령어를 이용하여 스토리지 블럭을 체크한다. 
disk는 150GB인 반면, root partition에는 100GB만 할당된 것을 확인할 수 있다.

```bash
ubuntu@my-ec2:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  150G  0 disk 
└─xvda1 202:1    0  100G  0 part /
```

growpart 명령어를 이용하여 파티션을 확장한다.

```bash
ubuntu@my-ec2:~$ sudo growpart /dev/xvda 1
CHANGED: partition=1 start=16065 old: size=209699102 end=209715167 new: size=314556702,end=314572767

ubuntu@my-ec2:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  150G  0 disk 
└─xvda1 202:1    0  150G  0 part /
```

resize2fs 명령을 이용하여 file system을 확장해주어야 마무리된다.

```bash
ubuntu@my-ec2:~$ sudo resize2fs /dev/xvda1
resize2fs 1.42.13 (17-May-2015)
Filesystem at /dev/xvda1 is mounted on /; on-line resizing required
old_desc_blocks = 7, new_desc_blocks = 10
The filesystem on /dev/xvda1 is now 39319587 (4k) blocks long.

# 확장이 잘 됬는지 확인
ubuntu@my-ec2:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            7.9G     0  7.9G   0% /dev
tmpfs           1.6G  169M  1.5G  11% /run
/dev/xvda1      148G   27G  115G  19% /
tmpfs           7.9G     0  7.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           7.9G     0  7.9G   0% /sys/fs/cgroup
tmpfs           1.6G     0  1.6G   0% /run/user/1000
```

Downtime없이 EC2 Instance의 Disk Volume을 늘이는데 성공했다. 

---

## Reference
- [https://support.amimoto-ami.com/english/self-hosting-accounts/increasing-your-ec2-volume-size](https://support.amimoto-ami.com/english/self-hosting-accounts/increasing-your-ec2-volume-size)
- [https://aws.amazon.com/ko/blogs/aws/amazon-ebs-update-new-elastic-volumes-change-everything/](https://aws.amazon.com/ko/blogs/aws/amazon-ebs-update-new-elastic-volumes-change-everything/)
- [https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/requesting-ebs-volume-modifications.html#elastic-volumes-limitations](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/requesting-ebs-volume-modifications.html#elastic-volumes-limitations)


