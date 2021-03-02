# REF
1. https://www.odoo.com/id_ID/forum/help-1/tag/tutorial-1040/questions
2. https://learnopenerp.blogspot.com/2017/12/python-function.html
3. https://stackoverflow.com/questions/12829608/text-align-class-for-inside-a-table
4. https://getbootstrap.com/docs/4.1/content/tables/
5. https://github.com/younismahsud/odoodiscussions/tree/master/openacademy/reports


# odooCode
source code note

# INHERITANCE
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
# DOMAIN
buat filter fieldnya berdasarkan kondisi tertentu atau nilai dari field tertentu
```bash
domain="[('name','=',Faris)]"
```
# COMPUTE FIELDs
melakukan komputasi saat itu juga bergantung dengan nilai suatu fields
```bash
name=Fields.Char(compute=_get_name)
@api.depends('....')
def _get_name(self):
        .....
       
```
# DEFAULT VALUES
```bash
user_id=m2o ('res.users',default= lambda self:self.env.user)
date=Fields.Date(default=Fields.Date.today())
active=Fields.Boolean(default=True)
```

# SELF ENV 
memberikan akses untuk request parameeter
```
self.env.cr atau self._cr >> database cursor object (untuk Querying DB)
self.env.uid atau self._uid >> current's user db id
self.env.user >> current's user record
self.env.context atau self._context >> context dict
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
