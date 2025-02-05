## Billing

### Public network fee

The EIPs bound to cloud product instances (such as CVM or NAT gateway) are free of charge. But you need to pay for public network traffic or public bandwidth when creating an EIP. For more information on the price, see the [pricing page](https://buy.cloud.tencent.com/price/cvm#tab0-list2). For more information on billing rules, see [purchasing public network](https://cloud.tencent.com/document/product/213/10579).

### Idle fee

For bare IPs not bound with any cloud product instances and bill-by-traffic EIPs, a resource occupation fee applies. See the below table: 

| Region of the EIP | Price per Hour for EIPs Without Bindings (If the duration is less than 1 hour, the EIP is billed on a pro rata basis) |
|---------|---------|
| Mainland China and Frankfurt | 0.20 CNY/hour |
| Hong Kong | 0.30 CNY/hour |
| North America and Western U.S. (Silicon Valley) | 0.25 CNY/hour |
| Singapore | 0.30 CNY/hour |

If the no-bindings duration is less than 1 hour, the EIP is billed **according to proportions**. For example, if an EIP is idle for 30 minutes, it is billed as `price per hour * 0.5` and is settled once an hour.

It is recommended to release Elastic Public IPs you no longer used to save your resources and costs. For more information on operation instructions, see [releasing EIPs](https://cloud.tencent.com/document/product/213/16586#.E9.87.8A.E6.94.BE.E5.BC.B9.E6.80.A7.E5.85.AC.E7.BD.91-ip).

## Arrears Processing
**For bare IPs, IPs billed by bandwidth on an hourly basis and IPs billed by traffic on an hourly basis:**

- If your account balance remains below 0 for more than **2** hours, EIPs will be retained for 24 hours free of charge. All EIP addresses will be unavailable in 24 hours, until the account balance has been topped up to an amount greater than 0.
- If your balance is still negative after the 24 + 2 hours, EIPs will be released.

