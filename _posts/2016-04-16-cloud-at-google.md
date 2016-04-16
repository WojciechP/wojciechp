---
title: "Cloud @ Google: Short Relation"
date: "2016-04-16"
categories: [events]
tags: [cloud, google, meetup]
excerpt: >
  Cloud @ Google - a few talks, lots of evangelising.
meta:
  description: "Relation from Cloud @ Google event"
  keywords: "cloud, google, GCP, web services"
image: "/images/gcp.png"
---


Cloud @ Google - a meeting focused on Google Cloud Platform - took place last
Thursday at [Google Warsaw Office]. There was a few short talks and quite a lot
of time for smaller table discussions.

# Cloud

From what the speakers said, GCP (Google Cloud Platform) seems to be extremely
important segment for Google, because they expect GCP's revenue to grow bigger
than Ads in five years. Therefore, lots of effort was put into spreading and
evangelising GCP.

GCP described by its creators is slightly different then well-known solutions
like Azure or AWS: the objective is to make users stop thinking about
infrastructure at all. So, mostly stressed feature of GCP was elastic scaling.
GCP is said to add extra VMs on the fly, launch the user app on each of them,
and adjust load balancer settings without any user intervention. This can be
obviously exploited for handling changes in workload, but Googlers also claim
that GCP can automagically relocate the VMs based on where the traffic is coming
from, so that the app servers are always close to the users.

Google is the last player to enter the "VM-as-a-service" market, but that isn't
necessarily a bad thing. Borg, the machine abstraction layer used by Google
internally to coordinate their own applications, works really well, so if GCP
provides a similar level of ease of use, I'm in.

# Vendors and Locks

Using that all auto-magic does not come free. GCP is heavily opinionated - it
should work best with other Google solutions for storage, queuing etc. Honestly,
getting a GCP VM just to have an instance of HBase there doesn't feel right. So
migrating to GCP is quite a vendor lock - but so is using AWS. I wonder whether
it's possible that one day all these services (like storage, queues, search or
autoscaling) will have some common API so that one could easily deploy an app in
either AWS or GCP. We'll see...


# Atmosphere

The meeting itself was incredibly well-prepared. Dozen Google employees working
on the GCP themselves were present and happily answered any questions and shared
their insights. After the talks, there was a round of table discussions not
necessarily related to GCP - I went in for "Creating Effective Engineering
Teams" to listen about processes and good practices Google uses. I was a bit
disappointed, though, because it all comes down to "we hire smart people,
provide them with tools and give them freedom - and they choose a process that
works for them". Not really helpful.

Overall, the meeting was great and I highly recommend going to a Google-hosted
event at least once.

  [Google Warsaw Office]: http://www.google.com/about/careers/locations/warsaw/
