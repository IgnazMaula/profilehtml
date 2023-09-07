Select ApproveJT, TRD_OH_Kode, sum(Nominal) Nominal, MOBL_IssuedDate, MOBL_MaturityDate From 
(SELECT ApproveJT, TRD_OH_Kode, MOBL_IssuedDate, MOBL_MaturityDate, Nominal = CASE WHEN nominalJual IS NULL THEN Nominal ELSE Nominal + nominalJual END FROM 
(SELECT approveJT, TRD_OH_Kode, Nominal, MOBL_IssuedDate, MOBL_MaturityDate, nominalJual = 
(SELECT SUM(Nominal) FROM TRD_OH_SBP_Sell WHERE noBeli = B.TRD_OH_NO) FROM TRD_OH_SBP B  
left join MOBL on MOBL_Code=TRD_OH_Kode where year(MOBL_MaturityDate)='2023' and month(MOBL_MaturityDate)='07'
) Y ) X group by ApproveJT, TRD_OH_Kode, MOBL_IssuedDate, MOBL_MaturityDate  order by MOBL_MaturityDate 
