--Surat Berharga Pemerintah
Select ApproveJatuhTempo, TRD_OH_Kode, MOBL_IssuedDate, MOBL_MaturityDate From TRD_SBPHTM
left join MOBL on MOBL_Code=TRD_OH_Kode 
where year(MOBL_MaturityDate)='2023' and month(MOBL_MaturityDate)='9'
group by ApproveJatuhTempo, TRD_OH_Kode, MOBL_IssuedDate, MOBL_MaturityDate  order by MOBL_MaturityDate
