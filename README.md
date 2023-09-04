select distinct TRD_REAL.bilyet as bilyet,trd_real.nomorsuratbank as nomorsuratbank from TRD_REAL 
left join TRD_D on NomorSuratBank=NoSuratBank 
left join MFBNK on TRD_D_Bank=Bank_ID and TRD_D_Branch=Branch_ID where Periode='2007/10' and DPCode='DP Smart' order by bilyet,nomorsuratbank asc
