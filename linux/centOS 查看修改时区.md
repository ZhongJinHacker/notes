```shell
// 查看时间各种状态，查看时区等
timedatectl

// 输出
Local time: 四 2014-12-25 10:52:10 CST
Universal time: 四 2014-12-25 02:52:10 UTC
RTC time: 四 2014-12-25 02:52:10
Timezone: Asia/Shanghai (CST, +0800)
NTP enabled: yes
NTP synchronized: yes
RTC in local TZ: no
```

  ```shell
// 列出所有时区
timedatectl list-timezones
// 将硬件时钟调整为与本地时钟一致, 0 为设置为 UTC 时间
timedatectl set-local-rtc 1
// 设置系统时区为上海
timedatectl set-timezone Asia/Shanghai
  ```

