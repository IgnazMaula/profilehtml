using System;

public class Converter
{
    public static string CNum(object tNum, byte tPrec = 30)
    {
        try
        {
            string cNum = "0";
            cNum = Math.Round(Convert.ToDecimal(tNum), tPrec).ToString();

            if (Format(0, "###.00").Substring(0, 1) == ",")
            {
                cNum = cNum.Replace(",", ".", 1, 1);
            }

            return cNum;
        }
        catch (Exception ex)
        {
            // Handle the error in an appropriate way
            Console.WriteLine(ex.Message);
            return null; // You might want to return something meaningful here
        }
    }

    // This function is used to mimic the Format behavior in VB
    public static string Format(object expression, string format)
    {
        // Implement your own logic to format the expression as needed
        throw new NotImplementedException();
    }
}
