tatic string CNum(object tNum, byte tPrec = 30)
    {
        try
        {
            string cNum = "0";
            cNum = Math.Round(Convert.ToDecimal(tNum), tPrec).ToString();
            
            if (string.Format("{0:###.00}", 0)[0] == ',')
            {
                cNum = cNum.Replace(",", ".", 1, 1);
            }
            
            return cNum;
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
            return null;
        }
    }
    
