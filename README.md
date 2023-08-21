var result = dbContext.TRD_D
    .Where(trd => trd.TRD_D_NoS == "1")
    .Join(
        dbContext.MFBNK,
        trd => new { trd.TRD_D_Bank, trd.TRD_D_Branch },
        mfbnk => new { Bank = mfbnk.Bank_ID, Branch = mfbnk.Branch_ID },
        (trd, mfbnk) => mfbnk.SubAcc
    );
