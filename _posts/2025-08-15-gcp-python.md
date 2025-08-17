---
layout: post
title: "Getting comfortable with GCP"
date: 2025-08-15
author: "Amit Indap"
tags:
  - Google Cloud Platform
  - Python
  - Docker
  - GitHub Actions
  - Gemini
---

I recently assumed the role of PTA President at my kids' school. It's my first experience with executive leadership and I'm quite excited. 
One of the goals of mine is to streamline PTA business processes, so they are less cumbersome and more efficient. One of the ways I've been doing this is building simple 
web applications with [Streamlit](https://streamlit.io/) and deploying them with [Cloud Run](https://cloud.google.com/run?hl=en) on [GCP](https://cloud.google.com/?hl=en). 

So far I've build Cloud Run apps. The first is for uploading and submitting reimbursements for PTA related purchases. The second is for volunteers to enter their hours. And the third is readng the Google 
sheet of volunteer hours and plotting data once every month as a scheduled job. Historically, I have deployed Streamlit apps on [Streamlit Community Cloud](https://streamlit.io/cloud), but I wanted
to grow my skill set and get famliar with more GCP services. 

To that end, I [vibe coded] the parts of the code dealing with the Google Cloud related APIs needed deploy the app on Cloud Run with [Gemini](https://gemini.google.com/app). This was my first 
time using Gemini, and I was quite happy. I wouldn't consider my self a cloud engineer, but I was able to pinpoint what [IAM roles](https://cloud.google.com/security/products/iam) on GCP I needed in order
to deploy my apps. 

Also, I've been getting into GitHub actions more and more lately, and was able to add workflows that would rebuild and deploy my containerized applications whenever I push a change to my repo. 

While all of my applications are fairly simple, they have a huge potential  to streamline how the PTA has operated in the past. For example, the previous year we have had to keep hard copies of receipts 
for reimbursements in a huge 3 ring binder. Now, we can store those in a cloud bucket and easily trace who submitted what and when. 

All this has made me a big fan of GCP, because I find the consol UI to be a lot simplier than AWS and I'm able learn new services farily quickly. 
And I'm pretty sure we are the only school in the district with a  [Github repo](https://github.com/hagepta) account. 
