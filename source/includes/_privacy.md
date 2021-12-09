# Data Privacy

While using every feature, AIbro needs data access at different levels. Usually, the more accesses a feature needs the better experience will be created (e.g. more time/money saving). Of course, whether using those features is totally upon your decision. By default, online_fit won't have any access. The following table gives a privacy item overview of each feature.

| Feature        | Server access | Model Access | Data Access | Reason                                                                                                                                          |
| -------------- | ------------- | ------------ | ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| Cooling server | **Yes**       | No           | No          | AIbro needs to retain the server access to reuse the cooling instances. The access will be permanently deleted once the instances is terminated |
| Stopped server | **Yes**       | No           | No          | AIbro needs to retain the server access to restart stopped instances. The access will be permanently deleted once the instances is terminated   |
| Report job     | No            | **Yes**      | **Yes**     | Support team needs the Model & data to replay the job and diagnose the reported issues                                                          |
