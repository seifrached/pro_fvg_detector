# 🚨 Trading Alert: Avoid CFDs 🚨

**_Warning_:**  
Trading Contracts for Difference (CFDs) can be extremely risky. Due to high leverage and volatility, CFDs can expose you to losses that exceed your initial investment—**potentially resulting in the loss of all your money**. 

**🔔 _Recommendation_:**  
For a safer trading environment, consider trading on **SPOT markets** (such as crypto spot or forex spot) where you are dealing with actual assets. SPOT trading generally involves lower leverage, which may reduce the risk of significant losses.  

**💡 _Reminder_:**  
- Always use proper risk management techniques.  
- Only trade with capital you can afford to lose.  
- Make sure to do thorough research or consult with a qualified financial advisor before making any trading decisions.

**_Disclaimer_:**  
This alert is provided for informational purposes only and does **not** constitute financial advice.




# PRO FVG Detector

## Overview
PRO_FVG_DETECTOR is a MetaTrader 4 (MQL4) custom indicator that automatically identifies, visualizes, and tracks Fair Value Gaps (FVGs) on price charts. The indicator detects both bullish and bearish FVGs, distinguishes between regular and "strong" FVGs, and monitors when FVGs are tested by subsequent price action.

#### Bullish FVG Screenshot

![Bullish FVG Screenshot](https://github.com/user-attachments/assets/be4c6231-39a2-4e2c-b472-659f033063f8)

#### Bearish FVG Screenshot

![Bearish FVG Screenshot](https://github.com/user-attachments/assets/5ddef9ff-376a-4ee0-9758-de26e99fa0c2)

## Core Functionality

### FVG Detection
- Scans price history to identify Fair Value Gaps based on specific candle patterns
- Detects two types of FVGs:
  - **Bullish FVGs**: When high[i+1] < low[i-1] with three consecutive bullish candles
  - **Bearish FVGs**: When low[i+1] > high[i-1] with three consecutive bearish candles
- Categorizes FVGs as "Strong" based on additional criteria:
  - Strong Bullish: When high[i+2] < open[i+1]
  - Strong Bearish: When low[i+2] > open[i+1]

### Visualization
- Draws rectangular objects on the chart to highlight FVG areas
- Uses customizable colors to distinguish between bullish/bearish and regular/strong FVGs
- Implements opacity settings for better visual clarity
- Adds informative text labels with detection time and FVG type

### FVG Testing & Management
- Monitors when price action tests an FVG (returns to the gap area)
- Changes the appearance of tested FVGs with different colors and reduced opacity
- Implements a countdown system that removes tested FVGs after a configurable delay
- Tracks all FVGs in an array structure for efficient management

### Configuration Options
- Flexible lookback period to control how much chart history to analyze
- Customizable colors for different FVG types
- Adjustable transparency/alpha settings
- Optional debug mode for troubleshooting
- Configurable removal delay for tested FVGs

## Technical Implementation
- Uses a structured approach with a custom FVGBOX data structure to track FVG properties
- Implements proper memory management by cleaning up objects when no longer needed
- Features proper initialization and deinitialization routines
- Includes time-based expiry mechanisms to manage chart objects efficiently

### License
This indicator is open-source. Please see the LICENSE file for details regarding permitted usage and distribution.

### Credits
Developed by [Seif Rached](https://github.com/seifrached). Contributions, suggestions, and improvements are welcome.
