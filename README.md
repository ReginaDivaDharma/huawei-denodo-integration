# HWC-Denodo-Integration
This is a summary of ways on how to integrate denodo with Huawei Cloud's Big Data System. Please do note this was made in early 2026, some things may have changed

# Background
Recently there was an urgent need to connect Denodo to Huawei Cloud Big Data Platform. But before we start what even is denodo? 
Denodo itself is actually a virtualization tool that acts as the layer of many data sources, whether on-prem or on-cloud from many ranges of vendors.
Denodo only acts as a virtualization layer, meaning it does not store any data whatsoever

# Problem
With the current knowledge, denodo supports JDBC protocol that can theoretically be extracted by huawei cloud's big data tools, this documentation shows you what works, and what didn't
