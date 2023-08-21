public byte UpdateJurnal(string tReffNumber, string tYear, double tValue, int tPeriodNumber, DateTime tDate, string tUser, string tDesc,
    string tCompany = "", string tJournalType = "DPPK", byte tUnposted = 1, byte tUnbalance = 0, string tCurr = "IDR", string tMD = "M",
    double tRate = 1, byte tExport = 0, string tBayarKepada = "", string tNilaiBayar = "0", string tBank = "", string tNoRekening = "")
{
    try
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

        string query = $"INSERT INTO GLTRHDSUM(GLTRHD_ReffNumb,GLTRHD_Company,GLTRHD_FiscalYear,GLTRHD_BaseAmmt,GLTRHD_CurrAmmt,GLTRHD_JournType," +
            $"GLTRHD_Period,GLTRHD_Unposted,GLTRHD_Unbalanced,GLTRHD_EntryDate,GLTRHD_LastEntryDate,GLTRHD_EmpID,GLTRHD_LastEmpID," +
            $"GLTRHD_TransactionDate,GLTRHD_Currency,GLTRHD_MD,GLTRHD_ExchangeRate,GLTRHD_UserID,GLTRHD_Export,GLTRHD_Note," +
            $"GLTRHD_UserDef2,GLTRHD_UserDef3,GLTRHD_UserDef6,GLTRHD_UserDef7) VALUES (" +
            $"'{tReffNumber}'," +
            $"'{EscapeString(tCompany)}'," +
            $"'{tYear}'," +
            $"'{Math.Round(tValue, 0)}'," +
            $"'{Math.Round(tValue, 0)}'," +
            $"'{tJournalType}'," +
            $"'{tPeriodNumber}'," +
            $"'{tUnposted}'," +
            $"'{tUnbalance}'," +
            $"'{tDate:yyyy/MM/dd}'," +
            $"'{tDate:yyyy/MM/dd}'," +
            $"'{tUser}'," +
            $"'{tUser}'," +
            $"'{tDate:yyyy/MM/dd}'," +
            $"'{tCurr}'," +
            $"'{tMD}'," +
            $"'{tRate}'," +
            $"'{tUser}'," +
            $"'{tExport}'," +
            $"'{EscapeString(tDesc)}'," +
            $"'{EscapeString(tBayarKepada)}'," +
            $"'{cNum(tNilaiBayar)}'," +
            $"'{EscapeString(tBank)}'," +
            $"'{EscapeString(tNoRekening)}')";
        
        // Execute the query using your database connection
        YourDatabaseConnection.Execute(query);

        return 1; // Assuming success
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.Message);
        return 0; // Assuming failure
    }
}

// Helper method to escape string values for SQL queries
private string EscapeString(string input)
{
    return input.Replace("'", "''");
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
