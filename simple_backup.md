# API process for a simple Backup

Before the backup we must register a new machine

## Machine registration
```PUT /register
HEADERS = {'kserver-name': 'autotest', 'kserver-key': 'test', 'Content-Length': 0, 'kserver-locale': 'test'}
REQUEST BODY = None
```

Return a UserId ( uid ) and a KKEY to authenticate in the following actions

For each Backup we must call the following endpoints.

## Synchro ( get local database of server state )
download file  https://dipsy.sante.kiwi-backup.com:4443/dbm?kkey=CA23fWvykxLdMk4FIaTCza1fzmtpaGgQ&uid=5ab781067da254000bd5a91b

## Start a new backup
```POST /start
REQUEST BODY = {'tz_dec': -7200, 'kkey': u'pbxJYAR06w8r9Iaovzx1MchTXWEHTl8m', 'uid': u'5ab76cd67da254000bd5a37b', 'conf': "[Config file content]"}
```

Return a bid ( backup indentifier ) Used in following call

For each File

## Send meta data ( about file metadata modification or directory entry ( no data stream ))
```
POST /meta
HEADERS = {'kserver-uid': u'5ab76cd67da254000bd5a37b', 'kserver-bid': u'5ab76cd77da254000bd5a37c'}
REQUEST BUFFER = [{"path": "/home/sebastien/kiwi/client4/kiwi-gui/server/testdata", "meta": "{\"st_ctime\": 1522007391, \"uid\": 1000, \"acl\": 509, \"gid\": 1000, \"st_size\": 4096, \"mode\": 16893, \"typ\": \"Dir\"}", "type": "Dir", "sha": ""}]
```
* The Request can be made for multiple file ( for performance reason )
* The return is in the same order. "Full" -> you should send the file.
...

## Send file stream ( simple )
```
PUT /full
HEADERS = {'kserver-uid': u'5ab801857da254000bd5aa98', 'kserver-sha': 'f16bdbd7e829b006689f1833453f20eabdea6b1b', 'kserver-gzipped': 1, 'kserver-path': 'L2hvbWUvc2ViYXN0aWVuL2tpd2kvY2xpZW50NC9raXdpLWd1aS9zZXJ2ZXIvdGVzdGRhdGEvdGVz\ndOKCrMW4wq0uZGF0\n', 'kserver-meta': '{"st_ctime": 1522008455, "st_mtime": 1522008455, "uid": 1000, "acl": 436, "gid": 1000, "st_size": 2097152, "mode": 33204, "typ": "File"}', 'kserver-bid': u'5ab801867da254000bd5aa99', 'kserver-type': 'File'}
REQUEST BODY = [CONTENT OF FILE]
```
The sha is the sha1 hash of the stream. It is verified on server and have to be exact. Used for deduplication


## End this backup
```POST /stop
HEADERS = None
REQUEST BODY = {'bid': u'5ab76cd77da254000bd5a37c', 'kkey': u'pbxJYAR06w8r9Iaovzx1MchTXWEHTl8m', 'uid': u'5ab76cd67da254000bd5a37b'}
```

## Synchro ( get local database of server state )
download file  https://dipsy.sante.kiwi-backup.com:4443/dbm?kkey=CA23fWvykxLdMk4FIaTCza1fzmtpaGgQ&uid=5ab781067da254000bd5a91b

## SEND Backup stats ( for the webadmin )
```
POST /stat
HEADERS = None
REQUEST BODY = {'tz_dec': -7200, 'uid': u'5ab781067da254000bd5a91b', 'ChangedFileSize': 0, 'bid': u'5ab781067da254000bd5a91c', 'ChangedFiles': 0, 'TimeSend': 0, 'NewFileSize': 0, 'NewFiles': 0, 'SendTarFileSize': 1139, 'TimeBackup': 1, 'VersionClient': 'Kclient 4.0.54 (Linux-3.13.0-143-generic-x86_64-with-Ubuntu-14.04-trustyx86_64)', 'SourceFileSize': 5120, 'SourceFiles': 2}
```
* All field must be present. Can be null except SourceFileSize .
* All size in octet

## RESTORE ( get tar stream of the asked file ( fid ) and descendent if recursive is true)

/tarmrestore?bid=0&kkey=CA23fWvykxLdMk4FIaTCza1fzmtpaGgQ&uid=5ab781067da254000bd5a91b&fid=5ab781087da254000bd5a925&recursive=True


