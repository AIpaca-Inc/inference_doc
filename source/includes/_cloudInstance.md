# Cloud Instance

We will use the word "machine" and "server" interchangeably with "instance" in the following content.

## Spot Vs On-demand Instance

### Machine id

By simply adding ".od" after machine id to convert instance type from spot to on-demand (e.g. _p2.xlarge_ is spot and _p2.xlarge.od_ is on-demand).

### Pricing

Spot instances are usually 70% cheaper than their corresponding on-demand instances.

![](https://drive.google.com/uc?export=view&id=1G6fVuxUu8Yofj_SBxl57m_lFZ83L4MzE)
![](https://drive.google.com/uc?export=view&id=1vomqhv0C7-ZjmwZ62dv3bjFJZtL2-wiT)

### Availability

As a tradeoff, spot instance requests are not always fulfilled. We defined the term **"availability"** as the success probability of
instance request.

Clearly, the availabilities of on-demand instances are always 100%. Therefore, it is guaranteed to get an on-demand instance as long as there is enough capacity in the AIbro marketplace. With a small possibility, AWS can run out of capacity itself, but it is not detectable until the request error occurs in jobs.

The availabilities of spot instances is varied by instance types and request time. In general, we found more powerful
instance types have less availability (e.g. p3.2xlarge is less available than p2.xlarge). Meanwhile, spot instances are
usually more available during non-working hours.

### Reliability

Spot instances have a chance to be interrupted by AWS. On-demand instances are always stable.
