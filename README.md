using System;
using System.Data.Linq;

public byte UpdateJurnal(string tReffNumber, string tYear, double tValue, int tPeriodNumber, DateTime tDate, string tUser, string tDesc,
    string tCompany = "", string tJournalType = "DPPK", byte tUnposted = 1, byte tUnbalance = 0, string tCurr = "IDR", string tMD = "M",
    double tRate = 1, byte tExport = 0, string tBayarKepada = "", string tNilaiBayar = "0", string tBank = "", string tNoRekening = "")
{
    try
    {
        using (YourDataContext dataContext = new YourDataContext()) // Replace YourDataContext with your actual data context class
        {
            double tDebet = 0;
            double tKredit = 0;
            if (tKredit < 0)
            {
                tDebet = -tKredit;
            }
            else if (tDebet < 0)
            {
                tKredit = -tDebet;
            }

            GLTRHDSUM newEntry = new GLTRHDSUM
            {
                GLTRHD_ReffNumb = tReffNumber,
                GLTRHD_Company = tCompany,
                GLTRHD_FiscalYear = tYear,
                GLTRHD_BaseAmmt = Math.Round(tValue, 0),
                GLTRHD_CurrAmmt = Math.Round(tValue, 0),
                GLTRHD_JournType = tJournalType,
                GLTRHD_Period = tPeriodNumber,
                GLTRHD_Unposted = tUnposted,
                GLTRHD_Unbalanced = tUnbalance,
                GLTRHD_EntryDate = tDate,
                GLTRHD_LastEntryDate = tDate,
                GLTRHD_EmpID = tUser,
                GLTRHD_LastEmpID = tUser,
                GLTRHD_TransactionDate = tDate,
                GLTRHD_Currency = tCurr,
                GLTRHD_MD = tMD,
                GLTRHD_ExchangeRate = tRate,
                GLTRHD_UserID = tUser,
                GLTRHD_Export = tExport,
                GLTRHD_Note = tDesc,
                GLTRHD_UserDef2 = tBayarKepada,
                GLTRHD_UserDef3 = cNum(tNilaiBayar),
                GLTRHD_UserDef6 = tBank,
                GLTRHD_UserDef7 = tNoRekening
            };

            dataContext.GLTRHDSUM.InsertOnSubmit(newEntry);
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
