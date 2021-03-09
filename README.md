# FinalYearProject
Final Year Project

# -*- coding: utf-8 -*-
"""
Created on Sun Mar  7 22:16:57 2021

@author: luke3
"""


import pandas as pd
import numpy as np
from datetime import date
import datetime
import time 
import random
import decimal
from numpy.random import normal
import matplotlib.pyplot as plt
import numpy_financial as npf

      
def create_df(count):    
    
    # uniform distribution of term in years
    years = 20 # random.randint(20,30)
    print("Term in years: " + str(years))
    
    payments_year = 12
        
    # normal distribution to calculate mortgage amount
    original_mortgage = 500000 #abs(np.round(normal(loc=400000, scale=150000),0)) #, size = 5000)
    print("Mortgage principal: €"+ str(original_mortgage))

    # start mortgage on 1st day of a random month between 2000 and 2005
    beginning_date = datetime.date(2000, 1, 1) 
    add_random_months = 36 #(random.randint(1,60))
    start_date = (beginning_date + pd.DateOffset(months=add_random_months)).to_pydatetime().date()
    print("Start date: " + str(start_date))
    end_date = (start_date + pd.DateOffset(years=years)).to_pydatetime().date()
    print("End date: " + str(end_date))     
    
    # pick a random number of years for the fixed period at 5%
    fixed_years = 2 #(random.randint(1,3))
    fixed_period_end = (start_date + pd.DateOffset(years=fixed_years)).to_pydatetime().date()
    print("Fixed period: " + str(fixed_years) + " years")
    print("End of fixed period: " + str(fixed_period_end))
    month_before_fixed_period_end = fixed_period_end - pd.DateOffset(months=1)
    print("Month before fixed period end " + str(month_before_fixed_period_end))
    print("Fixed period interest: 5%")
                   
    rng = pd.date_range(start_date, periods=(years * payments_year), freq='MS')
    rng.name = "Payment Date"
    df2 = pd.DataFrame(index=rng, columns=['Remaining period', 'Current balance', 'Interest Rate', 'Payment', 'Principal Paid', 'Interest Paid', 'To be paid', 'Ending Balance'], dtype='float')
    #df = pd.DataFrame(columns=['Payment', 'Principal Paid', 'Interest Paid', 'Ending Balance'], dtype='float')
    df2.reset_index(inplace=True)
    #df2.index = len(df2)
    df2.index += 1
    df2.index.name = "Period" 
    #df2.string = str(df2.index)
    print("Length of mortgage in months: " + str(len(df2)))
    #remaining_period2 = len(df2)
    remaining_period2 = ((int(len(df2))))
    
    df2['Remaining period'] = range(remaining_period2, 0, -1)
    
    end_of_fixed_to_end_of_mort = (end_date.year - fixed_period_end.year)*12 + (end_date.month - fixed_period_end.month)
    print("End of fixed period to end of mortgage: " + str(end_of_fixed_to_end_of_mort/12))
    
    int_rate_1_period = (end_date.year - date(2014, 1, 1).year)*12 + (end_date.month - date(2014, 1, 1).month)
    print("Length of first period in years: " + str(int_rate_1_period/12))
    
    int_rate_2_period = (end_date.year - date(2019, 1, 1).year)*12 + (end_date.month - date(2019, 1, 1).month)
    print("Length of second period in years: " + str(int_rate_2_period/12))
    
    end_of_break_to_end = (end_date.year - date(2020, 4, 1).year)*12 + (end_date.month - date(2020, 4, 1).month)
    print("End of break to end of mortgage: " + str(end_of_break_to_end/12))
        
     
    df2.loc[df2['Payment Date'] < str(fixed_period_end), 'Interest Rate'] = 5           
    df2.loc[(df2['Payment Date'] < '2014-1-1') & (df2['Payment Date'] >= str(fixed_period_end)), 'Interest Rate'] = 3
    df2.loc[(df2['Payment Date'] >= '2014-1-1') & (df2['Payment Date'] < '2019-1-1') & (df2['Payment Date'] > str(fixed_period_end)), 'Interest Rate'] = 2.15
    df2.loc[(df2['Payment Date'] >= '2019-1-1') & (df2['Payment Date'] > str(fixed_period_end)), 'Interest Rate'] = 1.15
    # https://datatofish.com/if-condition-in-pandas-dataframe/   
    
    df2.loc[1, 'Current balance'] = original_mortgage
    
    df2.loc[df2['Payment Date'] < str(fixed_period_end), 'Remaining months'] = years*12
    df2.loc[(df2['Payment Date'] < '2014-1-1') & (df2['Payment Date'] >= str(fixed_period_end)), 'Remaining months'] = end_of_fixed_to_end_of_mort
    df2.loc[(df2['Payment Date'] >= '2014-1-1') & (df2['Payment Date'] < '2019-1-1') & (df2['Payment Date'] > str(fixed_period_end)), 'Remaining months'] = (int_rate_1_period)
    df2.loc[(df2['Payment Date'] >= '2019-1-1') & (df2['Payment Date'] > str(fixed_period_end)), 'Remaining months'] = int_rate_2_period
    df2.loc[(df2['Payment Date'] >= '2020-4-1'), 'Remaining months'] = end_of_break_to_end
    
            
    interest = df2['Interest Rate']/100 
 
    remaining_years = df2['Remaining months']
    
    df2['Mortgage ID'] = count # gives each mortgage an ID
          
    df2["Int"] = 0    
        
    ###################################
    # fixed period
    
    df2.loc[(df2['Payment Date'] <= str(fixed_period_end)) , 'Principal'] = original_mortgage  
    mortgage = df2['Principal']  
        
    df2["Payment"] = -1 * npf.pmt(interest/12, remaining_years, mortgage) # df2["Principal Paid"] + df2["Interest Paid"]  
    df2["Interest Paid"] =  -1 * npf.ipmt(interest/12, df2.index, remaining_years, mortgage) #(df2["Ending Balance"])*(df2["Interest Rate"])
    df2["Principal Paid"] = -1 * npf.ppmt(interest/12, df2.index, remaining_years, mortgage) # df2["Payment"] - df2["Interest Paid"] 
    df2=df2.round(2)          

    df2["Ending Balance"] = 0.00
    df2.loc[1, "Ending Balance"] = original_mortgage - df2.loc[1, "Principal Paid"]
    df2
         
    #df2.loc[1, 'Int payment'] = original_mortgage*(0.05/12)

    for period in range(2, 25): #len(df2)+1):
        previous_balance = df2.loc[period-1, 'Ending Balance'].round(2)
        principal_paid = df2.loc[period, "Principal Paid"]
        
        if previous_balance == 0:
            df2.loc[period, ['Payment', 'Principal Paid', 'Interest Paid', 'Ending Balance']] == 0
            continue
        elif principal_paid <= previous_balance:
            df2.loc[period, 'Ending Balance'] = previous_balance - principal_paid  
            #df2.loc[period, 'Int payment'] = previous_balance*(0.05/12)
            df2.loc[period, 'Current balance'] = previous_balance.round(2)
            df2['Int'] = (df2["Current balance"]*(interest/12)).round(2)
            df2["Interest Paid"] = df2['Int']
            df2["To be paid"] = df2["Current balance"].round(2) + df2["Int"].round(2)
            df2["Ending Balance"] = df2["To be paid"] - df2["Payment"]

    df2["Principal Paid"] = df2["Payment"] - df2["Int"]    

    idx2 = (df2['Payment Date'] == str(month_before_fixed_period_end)) #'2004-12-01') #str(fixed_period_end)) #str(month_before_fixed_period_end))
    number2 = df2.loc[idx2, 'Ending Balance'].iat[0]
    print("Balance remaining at 2014-1-1 when interest changes to 3%: €"+ str(number2))
    df2.loc[(df2['Payment Date'] < '2014-1-1') & (df2['Payment Date'] >= str(fixed_period_end)) , 'Principal'] = number2
    

    #####################################
    # interest rate 1
       
    mortgage = df2['Principal'] 
      
    df2["Payment"] = -1 * npf.pmt(interest/12, remaining_years, mortgage) # df2["Principal Paid"] + df2["Interest Paid"]  
    df2["Interest Paid"] =  -1 * npf.ipmt(interest/12, df2.index, remaining_years, mortgage) #(df2["Ending Balance"])*(df2["Interest Rate"])
    df2["Principal Paid"] = -1 * npf.ppmt(interest/12, df2.index, remaining_years, mortgage) # df2["Payment"] - df2["Interest Paid"] 
    df2=df2.round(2)          

    for period in range(25, 133): #len(df2)+1):
        previous_balance = df2.loc[period-1, 'Ending Balance'].round(2)
        principal_paid = df2.loc[period, "Principal Paid"]
        
        if previous_balance == 0:
            df2.loc[period, ['Payment', 'Principal Paid', 'Interest Paid', 'Ending Balance']] == 0
            continue
        elif principal_paid <= previous_balance:
            df2.loc[period, 'Ending Balance'] = previous_balance - principal_paid  
            #df2.loc[period, 'Int payment'] = previous_balance*(0.05/12)
            df2.loc[period, 'Current balance'] = previous_balance.round(2)
            df2['Int'] = abs(df2["Current balance"]*(interest/12)).round(2)
            df2["Interest Paid"] = abs(df2['Int'])
            df2["To be paid"] = df2["Current balance"].round(2) + df2["Int"].round(2)
            df2["Ending Balance"] = df2["To be paid"] - df2["Payment"]

    df2["Principal Paid"] = df2["Payment"] - df2["Int"]    

    
    idx3 = (df2['Payment Date'] == '2013-12-01')
    number3 = df2.loc[idx3, 'Ending Balance'].iat[0]
    print("Balance remaining at 2014-1-1 when interest changes to 2.15%: " + str(number3))
    df2.loc[(df2['Payment Date'] >= '2014-1-1') , 'Principal'] = number3
    
    ############################
    # interest rate 2
    
    mortgage = df2['Principal'] 
      
    df2["Payment"] = -1 * npf.pmt(interest/12, remaining_years, mortgage) # df2["Principal Paid"] + df2["Interest Paid"]  
    df2["Interest Paid"] =  -1 * npf.ipmt(interest/12, df2.index, remaining_years, mortgage) #(df2["Ending Balance"])*(df2["Interest Rate"])
    df2["Principal Paid"] = -1 * npf.ppmt(interest/12, df2.index, remaining_years, mortgage) # df2["Payment"] - df2["Interest Paid"] 
    df2=df2.round(2)          

    for period in range(133, 193): #len(df2)+1):
        previous_balance = df2.loc[period-1, 'Ending Balance'].round(2)
        principal_paid = df2.loc[period, "Principal Paid"]
        
        if previous_balance == 0:
            df2.loc[period, ['Payment', 'Principal Paid', 'Interest Paid', 'Ending Balance']] == 0
            continue
        elif principal_paid <= previous_balance:
            df2.loc[period, 'Ending Balance'] = previous_balance - principal_paid  
            #df2.loc[period, 'Int payment'] = previous_balance*(0.05/12)
            df2.loc[period, 'Current balance'] = previous_balance.round(2)
            df2['Int'] = abs(df2["Current balance"]*(interest/12)).round(2)
            df2["Interest Paid"] = abs(df2['Int'])
            df2["To be paid"] = df2["Current balance"].round(2) + df2["Int"].round(2)
            df2["Ending Balance"] = df2["To be paid"] - df2["Payment"]

    df2["Principal Paid"] = df2["Payment"] - df2["Int"]
    
    idx4 = (df2['Payment Date'] == '2018-12-01')
    number4 = df2.loc[idx4, 'Ending Balance'].iat[0]
    print("Balance remaining at 2019-1-1 when interest changes to 1.15%: " + str(number4))
    df2.loc[(df2['Payment Date'] >= '2019-1-1') & (df2['Payment Date'] < '2020-4-1') , 'Principal'] = number4
    
        
    ##################################
    # interest rate 3
        
    mortgage = df2['Principal'] 
      
    df2["Payment"] = -1 * npf.pmt(interest/12, remaining_years, mortgage) # df2["Principal Paid"] + df2["Interest Paid"]  
    df2["Interest Paid"] =  -1 * npf.ipmt(interest/12, df2.index, remaining_years, mortgage) #(df2["Ending Balance"])*(df2["Interest Rate"])
    df2["Principal Paid"] = -1 * npf.ppmt(interest/12, df2.index, remaining_years, mortgage) # df2["Payment"] - df2["Interest Paid"] 
    df2=df2.round(2)        

    break1 = ((df2['Payment Date'] >= '2020-01-01') & (df2['Payment Date'] < '2020-04-01'))
    df2.loc[break1, 'Payment'] = 0

    
    for period in range(193, len(df2)+1):
    #while(df2['Payment Date'] >= '2019-1-1'):
        previous_balance = df2.loc[period-1, 'Ending Balance'].round(2)
        principal_paid = df2.loc[period, "Principal Paid"]
        
        if previous_balance == 0:
            df2.loc[period, ['Payment', 'Principal Paid', 'Interest Paid', 'Ending Balance']] == 0
            continue
        elif principal_paid <= previous_balance:
            df2.loc[period, 'Ending Balance'] = previous_balance - principal_paid  
            df2.loc[period, 'Current balance'] = previous_balance.round(2)
            df2['Int'] = abs(df2["Current balance"]*(interest/12)).round(2)
            df2["Interest Paid"] = abs(df2['Int'])
            df2["To be paid"] = df2["Current balance"].round(2) + df2["Int"].round(2)
            df2["Ending Balance"] = df2["To be paid"] - df2["Payment"]
        
    
    idx5 = (df2['Payment Date'] == '2020-3-01')
    number5 = df2.loc[idx5, 'Ending Balance'].iat[0]
    print("Balance remaining at 2020-3-1 break ends: " + str(number5))
    df2.loc[(df2['Payment Date'] >= '2020-4-1') , 'Principal'] = number5
    
    ####################
    # re-adjust for break
            
    mortgage = df2['Principal'] 
      
    df2["Payment"] = -1 * npf.pmt(interest/12, remaining_years, mortgage) # df2["Principal Paid"] + df2["Interest Paid"]  
    df2["Interest Paid"] =  -1 * npf.ipmt(interest/12, df2.index, remaining_years, mortgage) #(df2["Ending Balance"])*(df2["Interest Rate"])
    df2["Principal Paid"] = -1 * npf.ppmt(interest/12, df2.index, remaining_years, mortgage) # df2["Payment"] - df2["Interest Paid"] 
    df2=df2.round(2)        
    
    break1 = ((df2['Payment Date'] >= '2020-01-01') & (df2['Payment Date'] < '2020-04-01'))
    df2.loc[break1, 'Payment'] = 0
    
    for period in range(193, len(df2)+1):
    #while(df2['Payment Date'] >= '2019-1-1'):
        previous_balance = df2.loc[period-1, 'Ending Balance'].round(2)
        principal_paid = df2.loc[period, "Principal Paid"]
        
        if previous_balance == 0:
            df2.loc[period, ['Payment', 'Principal Paid', 'Interest Paid', 'Ending Balance']] == 0
            continue
        elif principal_paid <= previous_balance:
            df2.loc[period, 'Ending Balance'] = previous_balance - principal_paid  
            df2.loc[period, 'Current balance'] = previous_balance.round(2)
            df2['Int'] = abs(df2["Current balance"]*(interest/12)).round(2)
            df2["Interest Paid"] = abs(df2['Int'])
            df2["To be paid"] = df2["Current balance"].round(2) + df2["Int"].round(2)
            df2["Ending Balance"] = df2["To be paid"] - df2["Payment"]
        
            

            
    '''        
    idx5 = (df2['Payment Date'] == '2020-3-01')
    number5 = df2.loc[idx5, 'Ending Balance'].iat[0]
    print("Balance remaining at 2020-3-1 break ends: " + str(number5))
    df2.loc[(df2['Payment Date'] >= '2020-4-1') , 'Principal'] = number5

    
    ##################################
    # break readjusting pmt
        
    mortgage = df2['Principal'] 
      
    df2["Payment"] = -1 * npf.pmt(interest/12, remaining_years, mortgage) # df2["Principal Paid"] + df2["Interest Paid"]  
    df2["Interest Paid"] =  -1 * npf.ipmt(interest/12, df2.index, remaining_years, mortgage) #(df2["Ending Balance"])*(df2["Interest Rate"])
    df2["Principal Paid"] = -1 * npf.ppmt(interest/12, df2.index, remaining_years, mortgage) # df2["Payment"] - df2["Interest Paid"] 
    df2=df2.round(2)
    
               
    for period in range(207, len(df2)+1):
    #while(df2['Payment Date'] >= '2019-1-1'):
        previous_balance = df2.loc[period-1, 'Ending Balance'].round(2)
        principal_paid = df2.loc[period, "Principal Paid"]
        
        if previous_balance == 0:
            df2.loc[period, ['Payment', 'Principal Paid', 'Interest Paid', 'Ending Balance']] == 0
            continue
        elif principal_paid <= previous_balance:
            df2.loc[period, 'Ending Balance'] = previous_balance - principal_paid  
            df2.loc[period, 'Current balance'] = previous_balance.round(2)
            df2['Int'] = abs(df2["Current balance"]*(interest/12)).round(2)
            df2["Interest Paid"] = abs(df2['Int'])
            df2["To be paid"] = df2["Current balance"].round(2) + df2["Int"].round(2)
            df2["Ending Balance"] = df2["To be paid"] - df2["Payment"]
   ''' 
   
    number5 = df2.loc[len(df2)-1, 'Payment']    
    df2.loc[len(df2), 'Payment'] = number5
    df2.loc[len(df2), 'Current balance'] = number5
    ending_interest = df2.loc[len(df2)-1, 'Interest Rate']
    df2.loc[len(df2), 'Interest Paid'] = (number5*((ending_interest/12)/100)).round(2)
    df2.loc[len(df2), 'Int'] = df2.loc[len(df2), 'Interest Paid'] 
    
    df2.loc[len(df2), 'To be paid'] = df2.loc[len(df2)-1, 'Ending Balance'] + df2.loc[len(df2), 'Interest Paid'] 
    df2.loc[len(df2), 'Ending Balance'] = df2.loc[len(df2), 'To be paid'] - df2.loc[len(df2), 'Payment'] 
     
    df2["Principal Paid"] = df2["Payment"] - df2["Int"]
    
    df2.loc[len(df2), 'Principal Paid'] = number5 - (number5*((ending_interest/12)/100))
    #df2.loc[len(df2), 'Ending Balance'] = 0
       
    print("________________")
    return df2 

master_DF = create_df(1)
    
for x in range(0):
    df_two = create_df(x+2)
    #df_two.loc[len(df_two), "Payment"] == 0
    master_DF = master_DF.append(df_two, sort=False)
   
total_DF = pd.DataFrame((master_DF.groupby('Mortgage ID').sum()))
del total_DF["Ending Balance"]
del total_DF["Interest Rate"]
del total_DF["Remaining period"]
del total_DF["Current balance"]
del total_DF["To be paid"]
del total_DF["Remaining months"]
del total_DF["Int"]
del total_DF["Principal"]
print(total_DF)
total_DF.to_csv('Total_DF.csv')
master_DF.to_csv('PMT_DF.csv')



###############################################################################################
################## PART 2 ###############################################
### SAME DATAFRAME WITHOUT BREAK ####################

def create_df_without_break(count):    
    
    # uniform distribution of term in years
    years = 20 # random.randint(20,30)
    print("Term in years: " + str(years))
    
    payments_year = 12
        
    # normal distribution to calculate mortgage amount
    original_mortgage = 500000 #abs(np.round(normal(loc=400000, scale=150000),0)) #, size = 5000)
    print("Mortgage principal: €"+ str(original_mortgage))

    # start mortgage on 1st day of a random month between 2000 and 2005
    beginning_date = datetime.date(2000, 1, 1) 
    add_random_months = 36 #(random.randint(1,60))
    start_date = (beginning_date + pd.DateOffset(months=add_random_months)).to_pydatetime().date()
    print("Start date: " + str(start_date))
    end_date = (start_date + pd.DateOffset(years=years)).to_pydatetime().date()
    print("End date: " + str(end_date))     
    
    # pick a random number of years for the fixed period at 5%
    fixed_years = 2 #(random.randint(1,3))
    fixed_period_end = (start_date + pd.DateOffset(years=fixed_years)).to_pydatetime().date()
    print("Fixed period: " + str(fixed_years) + " years")
    print("End of fixed period: " + str(fixed_period_end))
    month_before_fixed_period_end = fixed_period_end - pd.DateOffset(months=1)
    print("Month before fixed period end " + str(month_before_fixed_period_end))
    print("Fixed period interest: 5%")
                   
    rng = pd.date_range(start_date, periods=(years * payments_year), freq='MS')
    rng.name = "Payment Date"
    df2 = pd.DataFrame(index=rng, columns=['Remaining period', 'Current balance', 'Interest Rate', 'Payment', 'Principal Paid', 'Interest Paid', 'To be paid', 'Ending Balance'], dtype='float')
    #df = pd.DataFrame(columns=['Payment', 'Principal Paid', 'Interest Paid', 'Ending Balance'], dtype='float')
    df2.reset_index(inplace=True)
    #df2.index = len(df2)
    df2.index += 1
    df2.index.name = "Period" 
    #df2.string = str(df2.index)
    print("Length of mortgage in months: " + str(len(df2)))
    #remaining_period2 = len(df2)
    remaining_period2 = ((int(len(df2))))
    
    df2['Remaining period'] = range(remaining_period2, 0, -1)
    
    end_of_fixed_to_end_of_mort = (end_date.year - fixed_period_end.year)*12 + (end_date.month - fixed_period_end.month)
    print("End of fixed period to end of mortgage: " + str(end_of_fixed_to_end_of_mort/12))
    
    int_rate_1_period = (end_date.year - date(2014, 1, 1).year)*12 + (end_date.month - date(2014, 1, 1).month)
    print("Length of first period in years: " + str(int_rate_1_period/12))
    
    int_rate_2_period = (end_date.year - date(2019, 1, 1).year)*12 + (end_date.month - date(2019, 1, 1).month)
    print("Length of second period in years: " + str(int_rate_2_period/12))
    

     
    df2.loc[df2['Payment Date'] < str(fixed_period_end), 'Interest Rate'] = 5           
    df2.loc[(df2['Payment Date'] < '2014-1-1') & (df2['Payment Date'] >= str(fixed_period_end)), 'Interest Rate'] = 3
    df2.loc[(df2['Payment Date'] >= '2014-1-1') & (df2['Payment Date'] < '2019-1-1') & (df2['Payment Date'] > str(fixed_period_end)), 'Interest Rate'] = 2.15
    df2.loc[(df2['Payment Date'] >= '2019-1-1') & (df2['Payment Date'] > str(fixed_period_end)), 'Interest Rate'] = 1.15
    # https://datatofish.com/if-condition-in-pandas-dataframe/   
    
    df2.loc[1, 'Current balance'] = original_mortgage
    
    df2.loc[df2['Payment Date'] < str(fixed_period_end), 'Remaining months'] = years*12
    df2.loc[(df2['Payment Date'] < '2014-1-1') & (df2['Payment Date'] >= str(fixed_period_end)), 'Remaining months'] = end_of_fixed_to_end_of_mort
    df2.loc[(df2['Payment Date'] >= '2014-1-1') & (df2['Payment Date'] < '2019-1-1') & (df2['Payment Date'] > str(fixed_period_end)), 'Remaining months'] = (int_rate_1_period)
    df2.loc[(df2['Payment Date'] >= '2019-1-1') & (df2['Payment Date'] > str(fixed_period_end)), 'Remaining months'] = int_rate_2_period
                
    interest = df2['Interest Rate']/100 
 
    remaining_years = df2['Remaining months']
    
    df2['Mortgage ID'] = count # gives each mortgage an ID
          
    df2["Int"] = 0    
        
    ###################################
    # fixed period
    
    df2.loc[(df2['Payment Date'] <= str(fixed_period_end)) , 'Principal'] = original_mortgage  
    mortgage = df2['Principal']  
        
    df2["Payment"] = -1 * npf.pmt(interest/12, remaining_years, mortgage) # df2["Principal Paid"] + df2["Interest Paid"]  
    df2["Interest Paid"] =  -1 * npf.ipmt(interest/12, df2.index, remaining_years, mortgage) #(df2["Ending Balance"])*(df2["Interest Rate"])
    df2["Principal Paid"] = -1 * npf.ppmt(interest/12, df2.index, remaining_years, mortgage) # df2["Payment"] - df2["Interest Paid"] 
    df2=df2.round(2)          

    df2["Ending Balance"] = 0.00
    df2.loc[1, "Ending Balance"] = original_mortgage - df2.loc[1, "Principal Paid"]
    df2
         
    #df2.loc[1, 'Int payment'] = original_mortgage*(0.05/12)

    for period in range(2, 25): #len(df2)+1):
        previous_balance = df2.loc[period-1, 'Ending Balance'].round(2)
        principal_paid = df2.loc[period, "Principal Paid"]
        
        if previous_balance == 0:
            df2.loc[period, ['Payment', 'Principal Paid', 'Interest Paid', 'Ending Balance']] == 0
            continue
        elif principal_paid <= previous_balance:
            df2.loc[period, 'Ending Balance'] = previous_balance - principal_paid  
            #df2.loc[period, 'Int payment'] = previous_balance*(0.05/12)
            df2.loc[period, 'Current balance'] = previous_balance.round(2)
            df2['Int'] = (df2["Current balance"]*(interest/12)).round(2)
            df2["Interest Paid"] = df2['Int']
            df2["To be paid"] = df2["Current balance"].round(2) + df2["Int"].round(2)
            df2["Ending Balance"] = df2["To be paid"] - df2["Payment"]

    df2["Principal Paid"] = df2["Payment"] - df2["Int"]    

    idx2 = (df2['Payment Date'] == str(month_before_fixed_period_end)) #'2004-12-01') #str(fixed_period_end)) #str(month_before_fixed_period_end))
    number2 = df2.loc[idx2, 'Ending Balance'].iat[0]
    print("Balance remaining at 2014-1-1 when interest changes to 3%: €"+ str(number2))
    df2.loc[(df2['Payment Date'] < '2014-1-1') & (df2['Payment Date'] >= str(fixed_period_end)) , 'Principal'] = number2
    

    #####################################
    # interest rate 1
       
    mortgage = df2['Principal'] 
      
    df2["Payment"] = -1 * npf.pmt(interest/12, remaining_years, mortgage) # df2["Principal Paid"] + df2["Interest Paid"]  
    df2["Interest Paid"] =  -1 * npf.ipmt(interest/12, df2.index, remaining_years, mortgage) #(df2["Ending Balance"])*(df2["Interest Rate"])
    df2["Principal Paid"] = -1 * npf.ppmt(interest/12, df2.index, remaining_years, mortgage) # df2["Payment"] - df2["Interest Paid"] 
    df2=df2.round(2)          

    for period in range(25, 133): #len(df2)+1):
        previous_balance = df2.loc[period-1, 'Ending Balance'].round(2)
        principal_paid = df2.loc[period, "Principal Paid"]
        
        if previous_balance == 0:
            df2.loc[period, ['Payment', 'Principal Paid', 'Interest Paid', 'Ending Balance']] == 0
            continue
        elif principal_paid <= previous_balance:
            df2.loc[period, 'Ending Balance'] = previous_balance - principal_paid  
            #df2.loc[period, 'Int payment'] = previous_balance*(0.05/12)
            df2.loc[period, 'Current balance'] = previous_balance.round(2)
            df2['Int'] = abs(df2["Current balance"]*(interest/12)).round(2)
            df2["Interest Paid"] = abs(df2['Int'])
            df2["To be paid"] = df2["Current balance"].round(2) + df2["Int"].round(2)
            df2["Ending Balance"] = df2["To be paid"] - df2["Payment"]

    df2["Principal Paid"] = df2["Payment"] - df2["Int"]    

    
    idx3 = (df2['Payment Date'] == '2013-12-01')
    number3 = df2.loc[idx3, 'Ending Balance'].iat[0]
    print("Balance remaining at 2014-1-1 when interest changes to 2.15%: " + str(number3))
    df2.loc[(df2['Payment Date'] >= '2014-1-1') , 'Principal'] = number3
    
    ############################
    # interest rate 2
    
    mortgage = df2['Principal'] 
      
    df2["Payment"] = -1 * npf.pmt(interest/12, remaining_years, mortgage) # df2["Principal Paid"] + df2["Interest Paid"]  
    df2["Interest Paid"] =  -1 * npf.ipmt(interest/12, df2.index, remaining_years, mortgage) #(df2["Ending Balance"])*(df2["Interest Rate"])
    df2["Principal Paid"] = -1 * npf.ppmt(interest/12, df2.index, remaining_years, mortgage) # df2["Payment"] - df2["Interest Paid"] 
    df2=df2.round(2)          

    for period in range(133, 193): #len(df2)+1):
        previous_balance = df2.loc[period-1, 'Ending Balance'].round(2)
        principal_paid = df2.loc[period, "Principal Paid"]
        
        if previous_balance == 0:
            df2.loc[period, ['Payment', 'Principal Paid', 'Interest Paid', 'Ending Balance']] == 0
            continue
        elif principal_paid <= previous_balance:
            df2.loc[period, 'Ending Balance'] = previous_balance - principal_paid  
            #df2.loc[period, 'Int payment'] = previous_balance*(0.05/12)
            df2.loc[period, 'Current balance'] = previous_balance.round(2)
            df2['Int'] = abs(df2["Current balance"]*(interest/12)).round(2)
            df2["Interest Paid"] = abs(df2['Int'])
            df2["To be paid"] = df2["Current balance"].round(2) + df2["Int"].round(2)
            df2["Ending Balance"] = df2["To be paid"] - df2["Payment"]

    df2["Principal Paid"] = df2["Payment"] - df2["Int"]
    
    idx4 = (df2['Payment Date'] == '2018-12-01')
    number4 = df2.loc[idx4, 'Ending Balance'].iat[0]
    print("Balance remaining at 2019-1-1 when interest changes to 1.15%: " + str(number4))
    df2.loc[(df2['Payment Date'] >= '2019-1-1'), 'Principal'] = number4
    
        
    ##################################
    # interest rate 3
        
    mortgage = df2['Principal'] 
      
    df2["Payment"] = -1 * npf.pmt(interest/12, remaining_years, mortgage) # df2["Principal Paid"] + df2["Interest Paid"]  
    df2["Interest Paid"] =  -1 * npf.ipmt(interest/12, df2.index, remaining_years, mortgage) #(df2["Ending Balance"])*(df2["Interest Rate"])
    df2["Principal Paid"] = -1 * npf.ppmt(interest/12, df2.index, remaining_years, mortgage) # df2["Payment"] - df2["Interest Paid"] 
    df2=df2.round(2)        


    
    for period in range(193, len(df2)+1):
    #while(df2['Payment Date'] >= '2019-1-1'):
        previous_balance = df2.loc[period-1, 'Ending Balance'].round(2)
        principal_paid = df2.loc[period, "Principal Paid"]
        
        if previous_balance == 0:
            df2.loc[period, ['Payment', 'Principal Paid', 'Interest Paid', 'Ending Balance']] == 0
            continue
        elif principal_paid <= previous_balance:
            df2.loc[period, 'Ending Balance'] = previous_balance - principal_paid  
            df2.loc[period, 'Current balance'] = previous_balance.round(2)
            df2['Int'] = abs(df2["Current balance"]*(interest/12)).round(2)
            df2["Interest Paid"] = abs(df2['Int'])
            df2["To be paid"] = df2["Current balance"].round(2) + df2["Int"].round(2)
            df2["Ending Balance"] = df2["To be paid"] - df2["Payment"]
        

   
    number5 = df2.loc[len(df2)-1, 'Payment']    
    df2.loc[len(df2), 'Payment'] = number5
    df2.loc[len(df2), 'Current balance'] = number5
    ending_interest = df2.loc[len(df2)-1, 'Interest Rate']
    df2.loc[len(df2), 'Interest Paid'] = (number5*((ending_interest/12)/100)).round(2)
    df2.loc[len(df2), 'Int'] = df2.loc[len(df2), 'Interest Paid'] 
    
    df2.loc[len(df2), 'To be paid'] = df2.loc[len(df2)-1, 'Ending Balance'] + df2.loc[len(df2), 'Interest Paid'] 
    df2.loc[len(df2), 'Ending Balance'] = df2.loc[len(df2), 'To be paid'] - df2.loc[len(df2), 'Payment'] 
     
    df2["Principal Paid"] = df2["Payment"] - df2["Int"]
    
    df2.loc[len(df2), 'Principal Paid'] = number5 - (number5*((ending_interest/12)/100))
    #df2.loc[len(df2), 'Ending Balance'] = 0
       
    print("________________")
    return df2 

master_DF2 = create_df_without_break(1)
    
for x in range(0):
    df_two2 = create_df_without_break(x+2)
    #df_two.loc[len(df_two), "Payment"] == 0
    master_DF2 = master_DF2.append(df_two2, sort=False)
   
total_DF2 = pd.DataFrame((master_DF2.groupby('Mortgage ID').sum()))
del total_DF2["Ending Balance"]
del total_DF2["Interest Rate"]
del total_DF2["Remaining period"]
del total_DF2["Current balance"]
del total_DF2["To be paid"]
del total_DF2["Remaining months"]
del total_DF2["Int"]
del total_DF2["Principal"]
print(total_DF2)
total_DF2.to_csv('Total_DF2.csv')
master_DF2.to_csv('PMT_DF2.csv')

print("OVERCHARGE AMOUNT:")
redress_df = pd.DataFrame(columns=['Payment', 'Interest Paid'])
redress_df['Payment'] = total_DF['Payment'] - total_DF2['Payment']
#redress_df['Principal Paid'] = total_DF['Principal Paid'] - total_DF2['Principal Paid']
redress_df['Interest Paid'] = total_DF['Interest Paid'] - total_DF2['Interest Paid']
print(redress_df)

