# HWC-Denodo Integration 🔗

> A summary of integration methods between **Denodo** and **Huawei Cloud's Big Data Platform**.
> 
> ⚠️ *This documentation was written in early 2026 — some details may have changed.*

---

## 📖 Background

**Denodo** is a data virtualization tool that acts as a unified layer over many data sources, whether on-premises or cloud-based, across a wide range of vendors. Importantly, Denodo only acts as a virtualization layer — **it does not store any data**.

---

## ❓ Problem

Denodo supports the **JDBC protocol**, which can theoretically be used by Huawei Cloud's Big Data tools to extract data. This documentation covers what worked, what didn't, and how to extract Denodo table views into the Huawei Cloud Big Data Platform.

---

## ✅ Solution

Two integration approaches were explored:

| Method | Status | Guide |
|---|---|---|
| Data Lake Insight (DLI) | ✅ Working | [View Guide](dli-integration.md) |
| Data Warehouse Service (DWS) | ⚠️ See notes | [View Guide](dws-integration.md) |

The general flow is:
1. Create a table view in Denodo.
2. Host Denodo with a **public IP**.
3. Connect Huawei Cloud's Big Data Platform to it via JDBC.

---

## 📚 Documentation

- [Denodo Server Setup](Denodo-server.md)
- [Data Preparation](data-preparation.md)
- [DLI Integration](dli-integration.md)
- [DWS Integration](dws-integration.md)
