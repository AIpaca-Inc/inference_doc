# Cloud Instance

The words "machine", "server", and “instance” are used interchangeably in the following content.

## Spot Vs On-demand Instance

### Machine id

By simply adding `".od"` after the machine ID, it is converted from a spot instance to an on-demand instance (e.g. _p2.xlarge_ is spot and _p2.xlarge.od_ is on-demand).

### Pricing

Spot instances are usually 70% cheaper than their corresponding on-demand instances.

![](https://drive.google.com/uc?export=view&id=1G6fVuxUu8Yofj_SBxl57m_lFZ83L4MzE)
![](https://drive.google.com/uc?export=view&id=1vomqhv0C7-ZjmwZ62dv3bjFJZtL2-wiT)

### Availability

As a tradeoff, spot instance requests are not always fulfilled. We define **"availability"** as the success probability of an instance request.

Clearly, the availability of on-demand instances is always 100%. Provided there is sufficient capacity in the Aibro marketplace, an on-demand instance is guaranteed. There is a small chance that AWS will reach capacity and create a bottleneck, although this cannot be detected until the request encounters an error during a job.

The availability of spot instances is varied by instance type and request time. We have found that in general, the more powerful instance types have less availability (e.g. p3.2xlarge is less available than p2.xlarge). Not surprisingly, we have also found that spot instances are more often available during non-business hours.

### Reliability

When choosing a configuration, it is relevant that spot instances can be interrupted by AWS, whereas on-demand instances are always stable.
