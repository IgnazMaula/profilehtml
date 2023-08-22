public byte UpdateDetailJurnal(string tReffNumber, int tLine, string tAcc, string tSubAcc, DateTime tDate, string tDesc, double tDebet, double tKredit,
    string tYear, string tPeriodNumber, string tJenisJurnal = "DPPK", string tEntity = "", string tProject = "", string tCurr = "IDR",
    double tRate = 1, string tModule = "", byte tUnposted = 1)
{
    try
    {
        if (tKredit < 0)
        {
            tDebet = -tKredit;
            tKredit = 0;
        }
        else if (tDebet < 0)
        {
            tKredit = -tDebet;
            tDebet = 0;
        }
        
        if (tDebet + tKredit == 0)
        {
            return 1;
        }

        tDesc = ConvertAcn(tAcc, "A") + " - " + ConvertAcn(tSubAcc, "S");

        using (YourDataContext dataContext = new YourDataContext()) // Replace YourDataContext with your actual data context class
        {
            GLTRANSUM newEntry = new GLTRANSUM
            {
                GLTRAN_Company = "", // Replace with the appropriate value
                GLTRAN_ReffNumb = tReffNumber,
                GLTRAN_LineNo = tLine,
                GLTRAN_AccountCode = tAcc,
                GLTRAN_JenisJurnal = tJenisJurnal,
                GLTRAN_SubAccount = tSubAcc,
                GLTRAN_TransDate = tDate,
                GLTRAN_Entity = tEntity,
                GLTRAN_Description = tDesc,
                GLTRAN_Project = tProject,
                GLTRAN_CurrencyCode = tCurr,
                GLTRAN_DebitAmount = cNum(tDebet),
                GLTRAN_CreditAmount = cNum(tKredit),
                GLTRAN_BaseDebetAmount = cNum(tDebet),
                GLTRAN_BaseCreditAmount = cNum(tKredit),
                GLTRAN_CuEffDate = tDate,
                GLTRAN_CuRate = tRate,
                GLTRAN_FiscalYear = tYear,
                GLTRAN_Module = tModule,
                GLTRAN_PeriodEntry = tPeriodNumber,
                GLTRAN_PeriodPost = tPeriodNumber,
                GLTRAN_Unposted = tUnposted
            };

            dataContext.GLTRANSUM.InsertOnSubmit(newEntry);
            dataContext.SubmitChanges();

            return 1; // Assuming success
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
        return 0; // Assuming failure
    }
}

// Helper method to convert string to numeric value
private double cNum(string input)
{
    double result;
    if (double.TryParse(input, out result))
    {
        return result;
    }
    return 0;
}


public string ConvertAcn(string acc, string Acct)
{
    try
    {
        using (YourDataContext dataContext = new YourDataContext()) // Replace YourDataContext with your actual data context class
        {
            if (Acct == "A")
            {
                var resultA = dataContext.MFCOA
                    .Where(item => item.MFCOA_Account == acc)
                    .Select(item => item.MFCOA_Des)
                    .FirstOrDefault();

                return resultA != null ? resultA.Trim() : "";
            }
            else
            {
                var resultS = dataContext.MFSOAM
                    .Where(item => item.MFSOAM_SubAccountNbr == acc)
                    .Select(item => item.MFSOAM_Description)
                    .FirstOrDefault();

                return resultS != null ? resultS.Trim() : "";
            }
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
        return "";
    }
}


