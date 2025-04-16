//+------------------------------------------------------------------+
//|                                             PRO_FVG_DETECTOR.mq4 |
//|                                                                  |
//|                        https://github.com/seifrached/            |
//+------------------------------------------------------------------+
#property copyright "Seif Rached - https://github.com/seifrached/"
#property link      "https://github.com/seifrached/"
#property version   "1.0"
#property description "PRO FVG DETECTOR"
#property strict
#property indicator_chart_window

// Input parameters
input bool   UseAllTimeChecking = false;  // Check all chart history instead of lookback
input int    LookBack = 1000;              // Number of candles to look back if not using all-time
input color  BullishFVGColor = C'144,238,144';  // Color for bullish FVG
input color  BearishFVGColor = C'255,182,193';  // Color for bearish FVG
input int    FVGAlpha = 50;               // Alpha/opacity level (0-255)
input color  StrongFVGBullish = C'0,128,0';     // Color for strong bullish FVG
input color  StrongFVGBearish = C'139,0,0';     // Color for strong bearish FVG
input bool   DebugMode = true;            // Enable debug messages
input color  TextBoxColor = clrBlack;     // Color for text inside Box
input int    RemovalDelayMinutes = 2;     // Delay in minutes before removing tested FVGs


// Structure for tracking FVG box
struct FVGBOX {
    string name;
    datetime expiry;      // Regular expiry time
    datetime testTime;    // When FVG was tested
    datetime removeTime;  // When FVG should be removed after testing
    bool tested;
    bool isBullish;
    bool isStrong;
};

FVGBOX fvgBoxes[];

//+------------------------------------------------------------------+
//| Custom indicator initialization function                         |
//+------------------------------------------------------------------+
int OnInit()
{
    Print("Enhanced FVG Detector initialized - Version 1.04");
    ArrayResize(fvgBoxes, 0);
    return(INIT_SUCCEEDED);
}

int GetRemovalDelay()
{
    return (Period() == PERIOD_M1) ? 20 : 120; 
    //return RemovalDelayMinutes * 60; // Convert minutes to seconds
}
//+------------------------------------------------------------------+
//| Calculate box expiry time                                        |
//+------------------------------------------------------------------+
datetime GetBoxExpiry()
{
    datetime currentTime = TimeCurrent();
    int periodMinutes = PeriodSeconds() / 60;
    return (currentTime - (currentTime % (periodMinutes * 60))) + (periodMinutes * 60);
}

datetime GetTestedExpiry(datetime testTime)
{
    return testTime + GetRemovalDelay();
}

//+------------------------------------------------------------------+
//| Get countdown string                                             |
//+------------------------------------------------------------------+
string GetCountdown(datetime endTime)
{
    int remaining = (int)(endTime - TimeCurrent());
    if(remaining <= 0) return "00:00";
    return StringFormat("%02d:%02d", remaining/60, remaining%60);
}

//+------------------------------------------------------------------+
//| Remove box from array                                           |
//+------------------------------------------------------------------+
void RemoveBoxFromArray(int index)
{
    if(index < 0 || index >= ArraySize(fvgBoxes)) return;
    
    for(int i = index; i < ArraySize(fvgBoxes) - 1; i++)
    {
        fvgBoxes[i] = fvgBoxes[i + 1];
    }
    ArrayResize(fvgBoxes, ArraySize(fvgBoxes) - 1);
}


//+------------------------------------------------------------------+
//| Custom indicator iteration function                              |
//+------------------------------------------------------------------+
int OnCalculate(const int rates_total,
                const int prev_calculated,
                const datetime &time[],
                const double &open[],
                const double &high[],
                const double &low[],
                const double &close[],
                const long &tick_volume[],
                const long &volume[],
                const int &spread[])
{
    if(rates_total < 4) return(0);
    
   
    //ObjectsDeleteAll(0, "FVG_*_TestLabel");
    CheckAndRemoveExpiredFVGs();
  
    datetime currentTime = TimeCurrent();
    datetime rightmostTime = Time[0] + PeriodSeconds()*50;
    
    
    int startBar = 2;
    int endBar = UseAllTimeChecking ? (rates_total - 3) : MathMin(startBar + LookBack, rates_total - 3);
    

    for(int i = endBar; i >= startBar; i--)
    {
       
        bool isBullishCandles = close[i+1] > open[i+1] && 
                               close[i] > open[i] && 
                               close[i-1] > open[i-1];
                               
        if(isBullishCandles && high[i+1] < low[i-1])
        {
            bool tested = false;
            datetime testTime = 0;
            
            for(int j = i-2; j >= 0; j--)
            {
                if(low[j] <= low[i-1])
                {
                    tested = true;
                    testTime = time[j];
                    break;
                }
            }
            
            string baseName = "FVG_Bull_" + TimeToString(time[i]);
            bool isStrong = high[i+2] < open[i+1];
            color fvgColor = isStrong ? StrongFVGBullish : BullishFVGColor;
            
            if(!tested)
            {
                if(ObjectCreate(0, baseName, OBJ_RECTANGLE, 0, time[i+1], high[i+1], rightmostTime, low[i-1]))
                {
                  
                    ObjectSetInteger(0, baseName, OBJPROP_COLOR, fvgColor);
                    ObjectSetInteger(0, baseName, OBJPROP_BGCOLOR, fvgColor);
                    ObjectSetInteger(0, baseName, OBJPROP_FILL, true);
                    ObjectSetInteger(0, baseName, OBJPROP_BACK, true);
                    ObjectSetInteger(0, baseName, OBJPROP_SELECTABLE, false);
                    ObjectSetInteger(0, baseName, OBJPROP_SELECTED, false);
                    ObjectSetInteger(0, baseName, OBJPROP_HIDDEN, false);
                    ObjectSetInteger(0, baseName, OBJPROP_ZORDER, 0);
                    int alpha = FVGAlpha;
                    color fillColor = (fvgColor & 0x00FFFFFF) | (alpha << 24);
                    ObjectSetInteger(0, baseName, OBJPROP_BGCOLOR, fillColor);
                    
                 
                    string labelName = baseName + "_Label";
                    datetime labelTime = rightmostTime - PeriodSeconds() * 2;
                    double priceMiddle = (high[i+1] + low[i-1]) / 2;
                    
                    if(ObjectCreate(0, labelName, OBJ_TEXT, 0, labelTime, priceMiddle))
                    {
                        string fvgType = isStrong ? "Strong Buy FVG" : "Buy FVG";
                        string labelText;
                        if(tested) {
                         labelText = "Tested FVG at: " + TimeToString(testTime, TIME_DATE|TIME_MINUTES) + " - " + fvgType;
                         } else {
                         labelText = "Detected at: " + TimeToString(time[i-1], TIME_DATE|TIME_MINUTES) + " - " + fvgType;
                         }
                        ObjectSetString(0, labelName, OBJPROP_TEXT, labelText);
                        ObjectSetInteger(0, labelName, OBJPROP_COLOR, TextBoxColor);
                        ObjectSetInteger(0, labelName, OBJPROP_FONTSIZE, 9);
                        ObjectSetString(0, labelName, OBJPROP_FONT, "Arial Bold");
                        ObjectSetInteger(0, labelName, OBJPROP_ANCHOR, ANCHOR_RIGHT);
                        ObjectSetInteger(0, baseName, OBJPROP_SELECTABLE, false);
                        ObjectSetInteger(0, baseName, OBJPROP_HIDDEN, false);
                        ObjectSetInteger(0, baseName, OBJPROP_BACK, false);
                    }
                    
                 
                    int size = ArraySize(fvgBoxes);
                    ArrayResize(fvgBoxes, size + 1);
                    fvgBoxes[size].name = baseName;
                    fvgBoxes[size].expiry = GetBoxExpiry();
                    fvgBoxes[size].tested = false;
                    fvgBoxes[size].testTime = 0;
                    fvgBoxes[size].removeTime = 0;  
                }
            }
            else if(tested && testTime != 0)
            {
                HandleTestedFVG(baseName, testTime, isStrong, true);
            }
        }
        
    
        bool isBearishCandles = close[i+1] < open[i+1] && 
                               close[i] < open[i] && 
                               close[i-1] < open[i-1];
                               
        if(isBearishCandles && low[i+1] > high[i-1])
        {
            bool tested = false;
            datetime testTime = 0;
            
            for(int j = i-2; j >= 0; j--)
            {
                if(high[j] >= high[i-1])
                {
                    tested = true;
                    testTime = time[j];
                    break;
                }
            }
            
            string baseName = "FVG_Bear_" + TimeToString(time[i]);
            bool isStrong = low[i+2] > open[i+1];
            color fvgColor = isStrong ? StrongFVGBearish : BearishFVGColor;
            
            if(!tested)
            {
                if(ObjectCreate(0, baseName, OBJ_RECTANGLE, 0, time[i+1], low[i+1], rightmostTime, high[i-1]))
                {
                   
                    ObjectSetInteger(0, baseName, OBJPROP_COLOR, fvgColor);
                    ObjectSetInteger(0, baseName, OBJPROP_BGCOLOR, fvgColor);
                    ObjectSetInteger(0, baseName, OBJPROP_FILL, true);
                    ObjectSetInteger(0, baseName, OBJPROP_BACK, true);
                    ObjectSetInteger(0, baseName, OBJPROP_SELECTABLE, false);
                    ObjectSetInteger(0, baseName, OBJPROP_SELECTED, false);
                    ObjectSetInteger(0, baseName, OBJPROP_HIDDEN, false);
                    ObjectSetInteger(0, baseName, OBJPROP_ZORDER, 0);
                    int alpha = FVGAlpha;
                    color fillColor = (fvgColor & 0x00FFFFFF) | (alpha << 24);
                    ObjectSetInteger(0, baseName, OBJPROP_BGCOLOR, fillColor);
                    
                 
                    string labelName = baseName + "_Label";
                    datetime labelTime = rightmostTime - PeriodSeconds() * 2;
                    double priceMiddle = (low[i+1] + high[i-1]) / 2 ;
                    
                    if(ObjectCreate(0, labelName, OBJ_TEXT, 0, labelTime, priceMiddle))
                    {
                        string fvgType = isStrong ? "Strong Sell FVG" : "Sell FVG";
                         string labelText;
                         if(tested) {
                             labelText = "Tested FVG at: " + TimeToString(testTime, TIME_DATE|TIME_MINUTES) + " - " + fvgType;
                         } else {
                             labelText = "Detected at: " + TimeToString(time[i-1], TIME_DATE|TIME_MINUTES) + " - " + fvgType;
                         }
                        ObjectSetString(0, labelName, OBJPROP_TEXT, labelText);
                        ObjectSetInteger(0, labelName, OBJPROP_COLOR, TextBoxColor);
                        ObjectSetInteger(0, labelName, OBJPROP_FONTSIZE, 9);
                        ObjectSetString(0, labelName, OBJPROP_FONT, "Arial Bold");
                        ObjectSetInteger(0, labelName, OBJPROP_ANCHOR, ANCHOR_RIGHT);
                        ObjectSetInteger(0, baseName, OBJPROP_SELECTABLE, false);
                        ObjectSetInteger(0, baseName, OBJPROP_HIDDEN, false);
                        ObjectSetInteger(0, baseName, OBJPROP_BACK, false);
                    }
                    
                   
                    int size = ArraySize(fvgBoxes);
                    ArrayResize(fvgBoxes, size + 1);
                    fvgBoxes[size].name = baseName;
                    fvgBoxes[size].expiry = GetBoxExpiry();
                    fvgBoxes[size].tested = false;
                    fvgBoxes[size].testTime = 0;
                    fvgBoxes[size].removeTime = 0;  
                }
            }
            else if(tested && testTime != 0)
            {
                HandleTestedFVG(baseName, testTime, isStrong, false);
            }
        }
    }
    
    ChartRedraw();
    return(rates_total);
}


//+------------------------------------------------------------------+
//| Handle FVG Testing                                               |
//+------------------------------------------------------------------+
void HandleTestedFVG(string baseName, datetime testTime, bool isStrong, bool isBullish)
{
   
    int boxIndex = -1;
    
 
    for(int k = 0; k < ArraySize(fvgBoxes); k++)
    {
        if(fvgBoxes[k].name == baseName)
        {
            boxIndex = k;
            break;
        }
    }
    
    if(boxIndex == -1)
    {
        boxIndex = ArraySize(fvgBoxes);
        ArrayResize(fvgBoxes, boxIndex + 1);
        fvgBoxes[boxIndex].name = baseName;
    }

    if(!fvgBoxes[boxIndex].tested)
    {
       
        fvgBoxes[boxIndex].tested = true;
        fvgBoxes[boxIndex].testTime = testTime;
        fvgBoxes[boxIndex].removeTime = testTime + (RemovalDelayMinutes * 60); 
        fvgBoxes[boxIndex].isBullish = isBullish;
        fvgBoxes[boxIndex].isStrong = isStrong;
        
        if(DebugMode)
        {
            Print("FVG Tested: ", baseName);
            Print("Test Time: ", TimeToString(testTime));
            Print("Will Remove at: ", TimeToString(fvgBoxes[boxIndex].removeTime));
        }
        
    
        color testedColor = isBullish ? (isStrong ? C'0,64,0' : C'144,178,144') : (isStrong ? C'69,0,0' : C'255,142,153');
        
        int testedAlpha = FVGAlpha / 2;
        color fillColor = (testedColor & 0x00FFFFFF) | (testedAlpha << 24);
        
        ObjectSetInteger(0, baseName, OBJPROP_BGCOLOR, fillColor);
        ObjectSetInteger(0, baseName, OBJPROP_COLOR, testedColor);
  
        string labelName = baseName + "_Label";
        if(ObjectFind(0, labelName) >= 0)
        {
            string fvgType = isBullish ? (isStrong ? "Strong Buy FVG" : "Buy FVG") : (isStrong ? "Strong Sell FVG" : "Sell FVG");
   
            string labelText = "TESTED " + fvgType + "\n " + "at: " + TimeToString(testTime, TIME_DATE|TIME_MINUTES) + "\n " + "Removes in: " + GetCountdown(fvgBoxes[boxIndex].removeTime);
            
            ObjectSetString(0, labelName, OBJPROP_TEXT, labelText);
            ObjectSetInteger(0, labelName, OBJPROP_COLOR, clrWhite); //ObjectSetInteger(0, labelName, OBJPROP_COLOR, isStrong ? clrWhite : C'128,128,128');
        }
    }
}

//+------------------------------------------------------------------+
//| Check and Remove Expired FVGs                                    |
//+------------------------------------------------------------------+
void CheckAndRemoveExpiredFVGs()
{
    datetime currentTime = TimeCurrent();
    
    for(int i = ArraySize(fvgBoxes) - 1; i >= 0; i--)
    {
        if(fvgBoxes[i].tested)
        {
            if(DebugMode)
            {
                Print("Checking FVG: ", fvgBoxes[i].name);
                Print("Current Time: ", TimeToString(currentTime));
                Print("Remove Time: ", TimeToString(fvgBoxes[i].removeTime));
                Print("Time Until Removal: ", fvgBoxes[i].removeTime - currentTime, " seconds");
            }
            
       
            if(currentTime >= fvgBoxes[i].removeTime)
            {
                if(DebugMode) Print("Removing FVG: ", fvgBoxes[i].name);
                
          
                ObjectDelete(0, fvgBoxes[i].name);
                ObjectDelete(0, fvgBoxes[i].name + "_Label");
                RemoveBoxFromArray(i);
            }
            else
            {
             
                string labelName = fvgBoxes[i].name + "_Label";
                if(ObjectFind(0, labelName) >= 0)
                {
                    string fvgType = fvgBoxes[i].isBullish ? 
                        (fvgBoxes[i].isStrong ? "Strong Buy FVG" : "Buy FVG") :
                        (fvgBoxes[i].isStrong ? "Strong Sell FVG" : "Sell FVG");
                        
                    string labelText = "TESTED " + fvgType + "\n " +"at : " + TimeToString(fvgBoxes[i].testTime, TIME_DATE|TIME_MINUTES) + "\n " +"Removes in: " + GetCountdown(fvgBoxes[i].removeTime);
                                     
                    ObjectSetString(0, labelName, OBJPROP_TEXT, labelText);
                    ObjectSetInteger(0, labelName, OBJPROP_COLOR, clrWhite); //ObjectSetInteger(0, labelName, OBJPROP_COLOR, fvgBoxes[i].isStrong ? clrWhite : C'128,128,128');
                }
            }
        }
    }
}

//+------------------------------------------------------------------+
//| Custom indicator deinitialization function                       |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
{
    ObjectsDeleteAll(0, "FVG");
}
