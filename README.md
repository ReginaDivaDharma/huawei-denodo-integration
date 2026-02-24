# HWC-Denodo-Integration
This is a summary of ways on how to integrate denodo with Huawei Cloud's Big Data System. Please do note this was made in early 2026, some things may have changed

# Background
Recently there was an urgent need to connect Denodo to Huawei Cloud Big Data Platform. But before we start what even is denodo? 
Denodo itself is actually a virtualization tool that acts as the layer of many data sources, whether on-prem or on-cloud from many ranges of vendors.
Denodo only acts as a virtualization layer, meaning it does not store any data whatsoever

# Problem
With the current knowledge, denodo supports JDBC protocol that can theoretically be extracted by huawei cloud's big data tools, this documentation shows you what works, and what didn't.
This documentation also aims to extract denodo's table views to huawei cloud big data platform.

# Solution
Initially we had two solution for the integration. One is using Data Warehouse Service (DWS) and the other which was more complicated, was using Data Lake Insight (DLI). The flow was quite simple, after creating a table view from denodo , we would host denodo with a public IP and the huawei cloud big data platform would try to connect to it. 
