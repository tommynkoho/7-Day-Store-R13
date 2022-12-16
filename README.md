# 7-Day-Store-R13

with Shelfout as (select distinct sos.store_number,count(so.sku_number) as Skus_Sent_to_Associates, thd_fiscal_week

from pr-store-merch-thd.STORE_BAY_INTEGRITY.SHELF_OUT as so

join pr-store-merch-thd.STORE_BAY_INTEGRITY.SHELF_OUT_SESSION sos

on sos.shelf_out_session_id = so.shelf_out_session_id

where source in('REACTIVE_PACKDOWN', 'ML_MODEL')

group by sos.store_number, thd_fiscal_week

order by sos.store_number, thd_fiscal_week), 

Part1 as (select store_number, Skus_Sent_to_Associates, MSH.RGN_NM as Regions, thd_fiscal_week

from Shelfout S

left join `analytics-met-thd.MET_BASE.MV_MET_STR_HIER` MSH 

on S.store_number = MSH.STR_NBR

group by store_number, Skus_Sent_to_Associates, Regions,thd_fiscal_week),

Worked as (select
       sos.store_number, count(sku_number) as Worked, thd_fiscal_week

 from pr-store-merch-thd.STORE_BAY_INTEGRITY.SHELF_OUT_QUESTION_ANSWER as soqa

 join pr-store-merch-thd.STORE_BAY_INTEGRITY.SHELF_OUT as so

       on soqa.shelf_out_id = so.shelf_out_id
  
join pr-store-merch-thd.STORE_BAY_INTEGRITY.SHELF_OUT_SESSION sos

      on so.shelf_out_session_id = sos.shelf_out_session_id

join pr-store-merch-thd.STORE_BAY_INTEGRITY.QUESTION_ANSWER qa

      on soqa.question_answer_id = qa.question_answer_id

join pr-store-merch-thd.STORE_BAY_INTEGRITY.ANSWER a

      on qa.answer_id = a.answer_id

join pr-store-merch-thd.STORE_BAY_INTEGRITY.QUESTION q
      on qa.question_id = q.question_id

where q.question = 'Was this SKU a shelf-out when you arrived?' and source in('REACTIVE_PACKDOWN', 'ML_MODEL')
and so.shelf_out_exclusion_reason_id is null
and (so.carry_fwd is not true or shelf_out_session like '%monday%')

group by sos.store_number, thd_fiscal_week),

Packdown as (select
       sos.store_number,
       count(sku_number) as Packdown, thd_fiscal_week

 from pr-store-merch-thd.STORE_BAY_INTEGRITY.SHELF_OUT_QUESTION_ANSWER as soqa

 join pr-store-merch-thd.STORE_BAY_INTEGRITY.SHELF_OUT as so

       on soqa.shelf_out_id = so.shelf_out_id
  
join pr-store-merch-thd.STORE_BAY_INTEGRITY.SHELF_OUT_SESSION sos

      on so.shelf_out_session_id = sos.shelf_out_session_id

join pr-store-merch-thd.STORE_BAY_INTEGRITY.QUESTION_ANSWER qa

      on soqa.question_answer_id = qa.question_answer_id

join pr-store-merch-thd.STORE_BAY_INTEGRITY.ANSWER a

      on qa.answer_id = a.answer_id

join pr-store-merch-thd.STORE_BAY_INTEGRITY.QUESTION q
      on qa.question_id = q.question_id

where q.question = 'Could you pack it down?'
   and a.answer = 'Yes'
   and source in('REACTIVE_PACKDOWN', 'ML_MODEL')
and so.shelf_out_exclusion_reason_id is null
and (so.carry_fwd is not true or shelf_out_session like '%monday%')

group by sos.store_number, thd_fiscal_week )

select Part1.*, Worked.Worked, Packdown.Packdown

from Part1

join Worked on 

Part1.store_number = Worked.store_number and

Part1.thd_fiscal_week = Worked.thd_fiscal_week

join Packdown on 

Part1.store_number = Packdown.store_number and
Part1.thd_fiscal_week = Packdown.thd_fiscal_week

group by Regions, store_number, Skus_Sent_to_Associates,Worked,Packdown,thd_fiscal_week   


![image](https://user-images.githubusercontent.com/17092274/208140980-1fcf79c4-1035-469b-a293-82d4608ba462.png)
