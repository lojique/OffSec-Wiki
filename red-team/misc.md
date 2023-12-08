# Misc

## Reversing a hex dump file

```
xdd -r fileName
```

## Zeek to analyze pcaps

Zeek (previously called _bro_) is a useful tool that enables high-level PCAP analysis at the application layer

{% embed url="https://youtu.be/O_z6o2xuvlw?t=1410" %}

```bash
zeek -Cr 0.pcap
less -S fileName.log
```

<figure><img src="../.gitbook/assets/image (590).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (336).png" alt=""><figcaption><p>By default zeek hides passwords</p></figcaption></figure>

```bash
# Changing the Zeek FTP Configuration to show passwords
sudo nano /usr/share/zeek/base/protocols/ftp/info.zeek
```

<figure><img src="../.gitbook/assets/image (328).png" alt=""><figcaption></figcaption></figure>

```bash
zeek -Cr 0.pcap
less -S fileName.log
```

<figure><img src="../.gitbook/assets/image (529).png" alt=""><figcaption></figcaption></figure>
