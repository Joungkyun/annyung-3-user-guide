Disk 파티션 정렬
==

## 1. 파티션 정렬의 이해

기본적으로 Disk partitioning 시에 첫번째 파티션은 항상 64번째 섹터에 해당하는 LBA 주소 63에서 시작하여, 이 섹터는 기본적으로 512 Byte의 크기를 가지고 있습니다.

이 이유는 CHS(Cylinder, Head, Sector)가 63 섹터 구조를 가지기 때문입니다. 디스크의 숨겨진 트랙(Logical Sector 0~62, Total 63 cector, 31.5KB) 영역이 끝나는 지점이 62 sector 이기 때문입니다.

이런 구조에서, 1 sector가 512 Byte일 경우에는 크게 문제가 없던 것이, disk가 고용량이 되면서, ***Advanced Format*** 형식(1sector 당 4K 할당)의 Disk가 나오면서 또는 RAID 구성시에 ***strip size***를 크게 잡을 경우에 문제가 발생하기 시작 합니다.


![](/assets/862fdcc34d6152ba7a4fda5858071a2f.jpg)
출처:https://blogs.msdn.microsoft.com/jimmymay/2009/05/08/disk-partition-alignment-sector-alignment-make-the-case-save-hundreds-of-thousands-of-dollars/

위의 그림은 RAID 구성시에 strip size을 64K 로 설정하고, cluster size를 64K 로 설정한 경우를 보여 줍니다.

첫번째 줄이 Strip size, 두번째 줄이 sector, 세번째 줄이 정령이 안된 파티션, 네번째 줄이 정령이 된 파티션을 의미 합니다.

위의 그림에서 정령이 되지 않은 3번째 파티션을 보면, sector size가 4K가 되면서 파티션이 63 sector에서 시작을 할 경우, RAID의 strip size 경계를 살짝 넘어서는 문제가 발생을 합니다. 이 의미는, 1번째 cluster를 처리를 하기 위하여 2개의 strip unit을 호출을 해야 한다는 의미가 됩니다. 즉 이 경계를 잘 맞추고 있다면 1번의 호출로 끝나는 것이 경계가 맞지 않아 2번 호출을 해야 한다는 것이 되고, 이 문제는 Disk 의 read/write 성능에 큰 손해를 끼치게 됩니다.

그래서 4번째 줄과 같이, 첫번째 파티션의 시작을 128 sector (64K)에서 시작을 하도록 권장을 하고 있습니다.

하지만, 대용량 파일의 read/write 성능을 위하여 format 시에 block size나 RIAD의 strip size를 1M로 설정하는 경우도 있기 때문에 요즘에는 128 sector 보다는 아래와 같이 2048 sector(1M)를 권장하고 있습니다.

![](/assets/800px-Partition-Alignment-LBA-Adressierung.png)
![](/assets/800px-Partition-Alignment-LBA-Adressierung-correct-aligned.png)
출처: https://www.thomas-krenn.com/en/wiki/Partition_Alignment

## 2. fdisk 를 이용한 파티셔닝

## 3. parted 를 이용한 파티셔닝

## 4. 파티션 정렬 확인