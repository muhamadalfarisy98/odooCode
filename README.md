# odooCode
source code note

```bash
def create_invoices(self):
        active_id = self._context.get('active_id')
        sale_id = self.env['sale.order'].browse(active_id)
        if sale_id.operating_unit_id:
            self = self.with_context(force_operating_unit=sale_id.operating_unit_id.id)   # self.with_context(force_operating_unit_id = sale_id.operating_unit_id.id)\
        return super(SaleAdvancePaymentInv, self).create_invoices()
```
- active_id =?
