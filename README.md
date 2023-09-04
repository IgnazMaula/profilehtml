select * from TRD_REAL left join TRD_D on NomorSuratBank=NoSuratBank 
left join MFBNK on TRD_D_Bank=Bank_ID and TRD_D_Branch=Branch_ID where DPCode='DP Smart' order by bilyet,nomorsuratbank asc
