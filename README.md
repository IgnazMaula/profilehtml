var result = 
    from trd in dbContext.TRD_D
    where trd.TRD_D_NoS == "1"
    join mfbnk in dbContext.MFBNK
    on new { Bank = trd.TRD_D_Bank, Branch = trd.TRD_D_Branch }
    equals new { Bank = mfbnk.Bank_ID, Branch = mfbnk.Branch_ID }
    into joinGroup
    from mfbnk in joinGroup.DefaultIfEmpty()
    select mfbnk.SubAcc;
