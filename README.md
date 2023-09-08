--Obligasi
Select ApproveJatuhTempo, TRD_OH_Kode, sum(SisaNominal) Nominal, MOBL_IssuedDate, MOBL_MaturityDate From TRD_HTM
left join MOBL on MOBL_Code=TRD_OH_Kode 
where year(MOBL_MaturityDate)='2021' and month(MOBL_MaturityDate)='10'
group by ApproveJatuhTempo, TRD_OH_Kode, MOBL_IssuedDate, MOBL_MaturityDate  order by MOBL_MaturityDate
