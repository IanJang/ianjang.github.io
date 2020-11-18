# 구동 중인 AWS EC2 디스크 용량 늘리기


## 들어가며

서비스 운영 중인 EC2의 디스크 용량이 가득 차는 문제가 발생할 수 있습니다. 이때 정지 시간(Downtime) 없이 디스크 용량을 늘릴 수 있습니다. 먼저 AWS Console 또는 AWS CLI를 이용하여 Volume 사이즈를 늘인 후 Volume이 연결된 EC2에 SSH 접속하여 Partition 확장을 해야 합니다. 더욱 자세한 내용은 본문을 참고 바랍니다.

## EC2 Volume Size 수정
아래 순서대로 작업을 진행합시다.
- AWS Console 접속 > Services > EC2 > Volumes 
- 수정하려는 볼륨 선택 > Actions - Modify Volume(볼륨수정) 선택 
- 수정을 원하는 Size(크기) 입력

작업을 완료하고 나면 Volume의 상태가 "in-use - optimizing"으로 변경됩니다. 이 상태가 "is-use completed"로 바뀌면 Partition 확장 작업을 진행합시다.

---

## Partition 확장

  먼저 아래와 같이 lsblk 명령어를 이용하여 스토리지 블록을 점검합니다. 결과를 보면 disk와 root partition의 용량이 서로 다릅니다. disk에는 150GB가 할당됐지만, root partition에는 100GB만 할당된 것을 확인할 수 있습니다.

```bash
ubuntu@my-ec2:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  150G  0 disk 
└─xvda1 202:1    0  100G  0 part /
```

  growpart 명령어를 이용하여 파티션을 확장합니다.

```bash
ubuntu@my-ec2:~$ sudo growpart /dev/xvda 1
CHANGED: partition=1 start=16065 old: size=209699102 end=209715167 new: size=314556702,end=314572767

ubuntu@my-ec2:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda    202:0    0  150G  0 disk 
└─xvda1 202:1    0  150G  0 part /
```

  resize2fs 명령을 이용하여 file system을 확장해주어야 마무리됩니다.

```bash
ubuntu@my-ec2:~$ sudo resize2fs /dev/xvda1
resize2fs 1.42.13 (17-May-2015)
Filesystem at /dev/xvda1 is mounted on /; on-line resizing required
old_desc_blocks = 7, new_desc_blocks = 10
The filesystem on /dev/xvda1 is now 39319587 (4k) blocks long.

# 확장이 잘 됐는지 확인
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

## 맺으며

  AWS를 이용하니 쉽고 간편한 작업을 통해 정지 시간(Downtime) 없이 EC2의 디스크 사이즈를 늘이는 데 성공했습니다. 이 글이 비슷한 문제를 겪는 분들에게 도움이 되었으면 합니다.
  
---

## Reference
- [https://support.amimoto-ami.com/english/self-hosting-accounts/increasing-your-ec2-volume-size](https://support.amimoto-ami.com/english/self-hosting-accounts/increasing-your-ec2-volume-size)
- [https://aws.amazon.com/ko/blogs/aws/amazon-ebs-update-new-elastic-volumes-change-everything/](https://aws.amazon.com/ko/blogs/aws/amazon-ebs-update-new-elastic-volumes-change-everything/)
- [https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/requesting-ebs-volume-modifications.html#elastic-volumes-limitations](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/requesting-ebs-volume-modifications.html#elastic-volumes-limitations)

