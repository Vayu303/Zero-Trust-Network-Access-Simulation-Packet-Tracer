## 1. AAA Server configuration

```
! Remove any previous configuration
no aaa authentication login default group radius local
no aaa authentication login SSH-AUTH group radius local
!
! Set up again the AAA configuration
aaa new-model
aaa authentication login default local
aaa authentication login SSH-AUTH group radius local
radius-server host 192.168.200.10 auth-port 1645 key radiuskey
!
! Explicit timeouts (doesn't work in PT)
radius-server timeout 5
radius-server retransmit 3
end
```

## 2. VTY lines configuration

```
configure terminal
line vty 0 4
 exec-timeout 5 0
 login authentication SSH-AUTH
 transport input ssh
line vty 5 15
 exec-timeout 5 0
 login authentication SSH-AUTH
 transport input ssh
end
```

## 3. Verify connection

```
! direct test for RADIUS authentication (doesn't work in PT)
test aaa group radius username password legacy
```

## 4. Configurazione Temporanea per Debug

```
configure terminal
!
! extended login
logging console 7
logging monitor 7
logging host 192.168.100.10
!
! enable RADIUS debug
debug radius
debug aaa authentication
```
