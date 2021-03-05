# ODOO
source code note

## REFERENSI
1. https://www.odoo.com/id_ID/forum/help-1/tag/tutorial-1040/questions
2. https://learnopenerp.blogspot.com/2017/12/python-function.html
3. https://stackoverflow.com/questions/12829608/text-align-class-for-inside-a-table
4. https://getbootstrap.com/docs/4.1/content/tables/
5. https://github.com/younismahsud/odoodiscussions/tree/master/openacademy/reports


## INHERITANCE
extend existing model in a modular way.
1. Modifikasi behavior nya
```bash
# menambahkan fitur, tersimpan di tabel yang sama
_name=obj1
_inherit=obj1 

# mengkopi model yang sudah ada, disimpan ditable new
_name=new
_inherit=obj1
```
2. Delegasi
menggabungkan beberapa inheritance, semisal
ada model mobils, terdiri atas inheritansi ban, casing mobil
```
multiple inheritance
```
## DOMAIN
buat filter fieldnya berdasarkan kondisi tertentu atau nilai dari field tertentu
```bash
domain="[('name','=',Faris)]"
```
## COMPUTE FIELDs
melakukan komputasi saat itu juga bergantung dengan nilai suatu fields
```bash
name=Fields.Char(compute=_get_name)
@api.depends('....')
def _get_name(self):
        .....
name = fields.Char(compute='_compute_name')
value = fields.Integer()

@api.depends('value')
def _compute_name(self):
     for record in self:
        record.name = "Record with value %s" % record.value
```
## DEFAULT VALUES
```bash
user_id=m2o ('res.users',default= lambda self:self.env.user)
date=Fields.Date(default=Fields.Date.today)
active=Fields.Boolean(default=True)
```

## SELF ENV 
memberikan akses untuk request parameeter
```
self.env.cr atau self._cr >> database cursor object (untuk Querying DB)
self.env.uid atau self._uid >> current's user db id
self.env.user >> current's user record
self.env.context atau self._context >> context dict
self.env['model_name']
```
## ONCHANGE
biasa dipakai buat raise error atau warning message jika input dari user ada yg ngaco
```bash
@api.onchange('amount', 'unit_price')
def _onchange_price(self):
    # set auto-changing field
    self.price = self.amount * self.unit_price
    # Can optionally return a warning and domains
    return {
        'warning': {
            'title': "Something bad happened",
            'message': "It was very bad indeed",
        }
    }
```
## CONSTRAINS
ngecek via python code kalau salah input, raise exception
```bash
@api.constrains('age')
def _check_something(self):
    for record in self:
        if record.age > 20:
            raise ValidationError("Your record is too old: %s" % record.age)
```

## ORM API
- Search method, return value = id model. mirip kaya select * from models where blablabl. search_count juga ada
```bash
self.env['res.partner'].search([['is_company', '=', True], ['customer', '=', True]])
res.partner(7, 18, 12, 14, 17, 19, 8, 31, 26, 16, 13, 20, 30, 22, 29, 15, 23, 28, 74)
```
- Browse method, return valuenya object (jadi bisa cek isi field2nya)
```bash
stud_obj=self.env['school_student'].browse(3,)
stud_obj.name
stud_obje.fields
```

## Override method ORM
- Create
```bash
def create(self,vals):
#vals dictionary
        vals['active']=True
        rtn=super(nama_class,self).create(vals)
        rtn.active=True #atau
        return rtn
```
- Write
dipakai saat update/edit record
```bash
def write(self,vals):
        rtn=super(nama_class,self).write(vals)
```
kalau mau edit
```bash
stud_list= self.env['school.student].browse([id])
stud_list.write({'active':True})
self._cr.commit() #update db
```

## Filter
return records in self satisfying func
```bash
# only keep records whose company is the current user's
records.filtered(lambda r: r.company_id == user.company_id)

# only keep records whose partner is a company
records.filtered("partner_id.is_company")
```


## SQL Execution
cr attribute on env is the cursor fot the current db transaction and allow executing SQL directly, for queries which are difficult to express using the ORM (complex join) or performance reasons.

```bash
self.env.cr.execute(query)
self._cr.commit()
        self.flush()
        query = """
            SELECT
                ppdr.id
            FROM
                product_pricelist_discount_regular ppdr
            WHERE
                ppdr.active=True
            """
        self.env.cr.execute(query)
        result = [x[0] for x in self._cr.fetchall()]
        result = self.env['product.pricelist.discount.regular'].sudo().browse(
            result)
        data = []
        for regular in result:
            data.append((
                regular.name,
                regular.payment_type,
                regular.min_quantity,
                regular.date_start,
                regular.date_end,
                regular.discount_percent,
                regular.count_partner,
                regular.applied_on,
            ))
        return data
```
## Manipulating Queries
- Where Query
```bash
where_query = ["am.state = 'posted'",
                       'NOT aml.exclude_from_invoice_tab']
date_from=2020
where_query.append("aml.date >= '%s'" % date_from)
where_query = 'WHERE ' + ' AND '.join(where_query)
# output:
WHERE am.state = 'posted' AND NOT aml.exclude_from_invoice_tab AND aml.date >= '2020'
```
- Implement where queries
```bash
query = """
            SELECT
                CASE WHEN cust.city IS NULL THEN '' ELSE cust.city END AS name,
                cust.city_id AS id,
                COALESCE(SUM(aml.price_subtotal * -aml.balance / ABS(aml.balance)),0) AS revenue
            FROM
                account_move_line aml
                JOIN account_move am ON am.id = aml.move_id AND am.move_type IN ('out_invoice','out_refund')
                JOIN product_product pp ON pp.id = aml.product_id
                JOIN res_users ru ON ru.id = am.invoice_user_id
                JOIN res_partner rp ON rp.id = ru.partner_id
                JOIN res_partner cust ON cust.id = aml.partner_id
                JOIN sale_order_line_invoice_rel solir ON solir.invoice_line_id = aml.id
            %(where_query)s
            GROUP BY
                cust.city_id,
                cust.city
        """ % {'where_query': where_query}
```

## Alternate fetching data
```bash
def get_data(self):
       appointments=self.env['..'].search([])
    for r in appointments:
        print(r.name) dll
```


## Hide Create/edit option di m2o field
di xml
```bash
<field name="" options="{'no_create_edit':True,'no_create':True,'no_open':True}"
```
## Nested Search
```bash
    [    '|'
         "&",
         ["start_datetime", "<",  '2020-01-07 09:00:00'],
         ["stop_datetime", ">=", '2020-01-07 09:00:00'],
         '|',
         "&",
         ["start_datetime", "<=", '2020-01-07 11:00:00'],
         ["stop_datetime", ">=", '2020-01-07 11:00:00'],
         '|',
         "&",
         ["start_datetime", ">=", '2020-01-07 09:00:00'],
         ["start_datetime", "<=",'2020-01-07 11:00:00'],
         "&",
         ["stop_datetime", ">=",  '2020-01-07 09:00:00'],
         ["stop_datetime", "<=",'2020-01-07 11:00:00']
     ] 

```
akan sama dengan
```bash
 (start_datetime <= '2020-01-07 09:00:00' and '2020-01-07 09:00:00' <= stop_datetime)
     or
    (start_datetime <= '2020-01-07 11:00:00' and '2020-01-07 11:00:00' <= stop_datetime)
    or 
    ('2020-01-07 09:00:00' <= start_datetime and start_datetime <= '2020-01-07 11:00:00') 
    or
    ('2020-01-07 09:00:00' <= stop_datetime and stop_datetime <= '2020-01-07 11:00:00')

```
## button icon
![btn](https://user-images.githubusercontent.com/23287190/110078270-1256ce80-7dba-11eb-8e68-7632ec21022a.png)
```bash
<button name="action_download_button" string="Print" type="object" icon="fa-print" />
```



## DUMMY TEST
```bash
def create_invoices(self):
        active_id = self._context.get('active_id')
        sale_id = self.env['sale.order'].browse(active_id)
        if sale_id.operating_unit_id:
            self = self.with_context(force_operating_unit=sale_id.operating_unit_id.id)   # self.with_context(force_operating_unit_id = sale_id.operating_unit_id.id)\
        return super(SaleAdvancePaymentInv, self).create_invoices()
```
- active_id =?


dummy testing [1]
<button name="tes_dummy" type="object" string="Tesing"/>
```bash
def tes_dummy(self):
        print('count',self.student_profile_ids)
        print('count1',type(self.student_profile_ids))
        print('len',len(self.student_profile_ids))
        return {
                'effect':{
                        'fadeout':'slow',
                        'message':'Testing..',
                        'type':'rainbow_man',
                }

        }
        # print('count3',type(self.student_profile_ids.id))
        # print('tes',type(int((self.env['student.profile'].search_count([])))))
        # print('tes',(self.env['student.profile'].search_count([])))
        # print('tes',len(self.env['student.profile'].search([]).mapped('id') ))
        # print('id tag',self.env['school.class'].browse(1).name)
        # print('active id',self.env['school.class'].browse(self._context.get('active_ids', [])).name  )
        # print('active id2',self.env['school.class'].browse(self._context.get('active_ids', []))  )
        # print('ref',self.env['school.class'].ref('website_shahaba./'))
        
        # print('tes2',len(self.student_profile_ids.id))
        # print('tes3',self.student_profile_ids.id)
        # print('tes4',(self.student_profile_ids.name))
##REVISI
```

## default get
```bash
@api.model 
    def default_get(self, field_list=[]):
        monitor_obj=self.env['monitor.tahfidz']
        print('searching',monitor_obj.search([]))
        monitor_list=monitor_obj.search([])
        print('read',monitor_list.read([]))
        print('field_list',field_list)    
        rtn= super(MonitorTahfidz,self).default_get(field_list)
        print('rtn',rtn)
        rtn['ayat_selesai']='3'
        print('rtn update',rtn)
        return rtn
    # @api.model 
    # def create(self,vals):
    #     print('create vals',vals)
    #     rtn=super(MonitorTahfidz,self).create(vals)
    #     print('create rtn',rtn)
    #     return rtn
    # @api.model
    # def write(self,vals):
    #     print('write vals',vals)
    #     print('write self',self)
    #     rtn=super(MonitorTahfidz,self).create(vals)
    #     print('write rtn',rtn)
    #     return rtn
```

## constrain

```bash
    # #in case terjadi sql_constraint    
    # def copy(self, default=None):
    #     default = dict(default or {})

    #     copied_count = self.search_count(
    #         [('subject_name', '=like', u"Copy of {}%".format(self.subject_name))])
    #     if not copied_count:
    #         new_subject_name = u"Copy of {}".format(self.subject_name)
    #     else:
    #         new_subject_name = u"Copy of {} ({})".format(self.subject_name, copied_count)

    #     default['subject_name'] = new_subject_name
    #     return super(SubjectPelajaran ,self).copy(default)



    # _sql_constraints= [
    #     ('check_name_description','CHECK(subject_name!=description)',
    #     "nama mata pelajaran seharusnya berbeda dengan deskripsi"),
    #     ('name_unique','UNIQUE(subject_name)',"mata pelajaran harus unik")
    # ]
```

## calendar

```bash
 """TESTING durasi ujian"""
    # duration = fields.Float(digits=(6,2), help="duration in days")
    # end_date = fields.Date(string='End date', store=True,compute='_get_end_date',inverse='_set_end_date')
    
    #menambah form kalendar dengan lama durasi waktu mulai ditambah dengan durasi nya
    #atau dapat di move via drag and drop (inverse attribute)
    # @api.depends('start_date', 'duration')
    # def _get_end_date(self):
    #     for r in self:
    #         if not (r.start_date and r.duration):
    #             r.end_date = r.start_date
    #             continue

    #         # Add duration to start_date, but: Monday + 5 days = Saturday, so
    #         # subtract one second to get on Friday instead
    #         duration = timedelta(days=r.duration, seconds=-1)
    #         r.end_date = r.start_date + duration

    # def _set_end_date(self):
    #     for r in self:
    #         if not (r.start_date and r.end_date):
    #             continue

    #         # Compute the difference between dates, but: Friday - Monday = 4 days,
    #         # so add one day to get 5 days instead
    #         r.duration = (r.end_date - r.start_date).days + 1


```

## calendar views xml
```bash
<!-- jadwal.ujian calendar view -->
        <!-- <record id="jadwal_ujian_view_calendar" model="ir.ui.view">
            <field name="name">jadwal.ujian.view.calendar</field>
            <field name="model">jadwal.ujian</field>
            <field name="arch" type="xml">
                <calendar string="Session Calendar" date_start="start_date" date_stop="end_date" color="subject_id">
                    <field name="semester"/>
                </calendar>
            </field>
        </record> -->

```


## report
```bash
<!-- <div class="row">
                        <div class="col-8"> 1. MOHON PEMBAYARAN DI TRANSFER KE REKENING</div>
                        <div class="col-4">HORMAT KAMI,</div>
                    </div> -->

                    <!-- <div class="col-4" style="border: 1px solid black;">HORMAT KAMI,</div> -->
                    <!-- <p class="col-6 bm-6">  </p>
                    <p class="col-2 bm-6">hormat kami,  </p>


                    <h3 class="text-center">This is the 3 equal columns divided as 12/3</h3>
                    <div class="row">
                        <div class="col-3"></div>
                        <div class="col-3"></div>
                        <div class="col-3" ></div>
                        <div class="col-3" >hormat kami</div>
                    </div> -->


                    <div id="total" class="row justify-content-end">
                        <div class="col-4">
                            <table class="table table-sm">
                                <tr class="border-black">
                                    <td name="td_subtotal_label"><strong>Subtotal</strong></td>
                                    <td class="text-right">
                                        <span t-esc="doc.total_amount"
                                        t-options='{"widget": "monetary", "display_currency": doc.invoice_ids.currency_id}'/>
                                    </td>
                                </tr>
                            </table>
                        </div>
                    </div>

```
