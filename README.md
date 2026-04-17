# HWC-Denodo Integration 🔗

> A summary of integration methods between **Denodo** and **Huawei Cloud's Big Data Platform**.
> 
> ⚠️ *This documentation was written in early 2026 some details may have changed.*

---

## 📖 Background

**Denodo** is a data virtualization tool that acts as a unified layer over many data sources, whether on-premises or cloud-based, across a wide range of vendors. Importantly, Denodo only acts as a virtualization layer **it does not store any data**. So the only way we can connect to it is by using the JDBC protocol.
 
---

## ❓ Problem

Denodo supports the **JDBC protocol**, which can theoretically be used by Huawei Cloud's Big Data tools to extract data. This documentation covers what worked, what didn't, and how to extract Denodo table views into the Huawei Cloud Big Data Platform.

---

## ✅ Solution

Two integration approaches were explored:

| Method | Status | Guide |
|---|---|---|
| Data Lake Insight (DLI) | ✅ Working | [View Guide](dli-integration.md) |
| Data Warehouse Service (DWS) | ⚠️ See notes | DWS does not have a JDBC driver |

The general flow is:
1. [Prepare your data](data-preparation.md)
2. [Prepare your denodo](Denodo-server.md)
3. [DLI Integration](dli-integration.md)
4. [DWS Integration (This is not possible)](dws-integration.md)
