
Common formats:

- Backblaze B2: `b2:bucket` or `b2:bucket/prefix`
- AWS S3: `s3:bucket` or `s3:bucket/prefix`
- Google Cloud: `gs:bucket:/` or `gs:bucket:/prefix`
- SFTP: `sftp:user@host:/path/to/repo`
- Local: `/mnt/backupdisk/repo1`
- Rclone: `rclone:remote:path` (requires rclone installation)

- **Environment Variables** Storage provider credentials:
    - S3: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
    - B2: `B2_ACCOUNT_ID`, `B2_ACCOUNT_KEY`
    - Google Cloud: `GOOGLE_PROJECT_ID`, `GOOGLE_APPLICATION_CREDENTIALS`

https://github.com/garethgeorge/backrest/releases


### [Backup API](https://garethgeorge.github.io/backrest/docs/api#backup-api)

The backup API can be used to trigger execution of a plan e.g.

```
curl -X POST 'localhost:9898/v1.Backrest/Backup' --data '{"value": "YOUR_PLAN_ID"}' -H 'Content-Type: application/json'
```

### [Operations API](https://garethgeorge.github.io/backrest/docs/api#operations-api)

The operations API can be used to fetch operation history e.g.

```
curl -X POST 'localhost:9898/v1.Backrest/GetOperations' --data '{}' -H 'Content-Type: application/json'
```

More complex selectors can be applied e.g.

```
curl -X POST 'localhost:9898/v1.Backrest/GetOperations' --data '{"selector": {"planId": "YOUR_PLAN_ID"}}' -H 'Content-Type: application/json'
```

For details on the structure of operations returned see the [operations.proto](https://github.com/garethgeorge/backrest/blob/main/proto/v1/operations.proto).