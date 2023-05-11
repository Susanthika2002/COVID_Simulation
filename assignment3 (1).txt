import pandas as pd
import numpy as np
import random as rm
from sim_parameters import TRASITION_PROBS as t
from sim_parameters import HOLDING_TIMES as h
import csv
from helper import create_plot


def run(countries_csv_name,countries, sample_ratio,start_date,end_date):
    df=pd.read_csv(countries_csv_name)
    r=np.floor(df['population']/sample_ratio)
    df['less_5']=np.floor(r*df['less_5']/100).astype('int')
    df['5_to_14']=np.floor(r*df['5_to_14']/100).astype('int')
    df['15_to_24']=np.floor(r*df['15_to_24']/100).astype('int')
    df['25_to_64']=np.floor(r*df['25_to_64']/100).astype('int')
    df['over_65']=np.floor(r*df['over_65']/100).astype('int')

    states = ["H","I","S","D","M"]
    dates=pd.date_range(start_date,end_date)
    def Markov(age,days=len(dates)):
       simu=[]
       n=list(h[age].values())
       m=list(t[age].values())
       transitionName=[]
       transitionMatrix=[]
       for i in range(len(m)):
          transitionName.append(list(m[i].keys()))
          transitionMatrix.append(list(m[i].values()))
       activityToday = "H"
       activityYesterday="H"
       activityList = [activityToday]
       prevstate=[]
       i = 0
       c=[activityToday]
       f=[n[states.index(activityToday)]]

       while i < days:
          change = np.random.choice(transitionName[states.index(activityToday)],replace=True,p=transitionMatrix[states.index(activityToday)])
          activityList.append(change)
          #activityYesterday=activityToday
          activityToday=change
          i=i+n[states.index(activityToday)]
          if n[states.index(activityToday)] == 0:
             i+=1
             prevstate.append(c[-1])
             c.append(change)
             f.append(n[states.index(activityToday)])
          for _ in range(n[states.index(activityToday)]):
             prevstate.append(c[-1])
             c.append(change)
             f.append(n[states.index(activityToday)])
       simu.append(c[:days])
       simu.append(f[:days])
       simu.append(prevstate[:days])
       return simu

    simulated=pd.DataFrame(columns=['id','age_group_name','country','date','state','staying_days','prev_state'])
    sumar=pd.DataFrame(columns=['date','country','D','H','I','M','S'])
    c=-1
    for i in range(df.shape[0]):
        if df.iloc[i]['country'] in countries:
            li=[]
            rsim=pd.DataFrame(columns=['id','age_group_name','country','date','state','staying_days','prev_state'])
            sim=pd.DataFrame(columns=['id','age_group_name','country','date','state','staying_days','prev_state'])
            su=pd.DataFrame(columns=['date','country','D','H','I','M','S'])
            li.extend([df.iloc[i]['less_5'],df.iloc[i]['5_to_14'],df.iloc[i]['15_to_24'],df.iloc[i]['25_to_64'],df.iloc[i]['over_65']])
            for j in range(len(li)):
                for k in range(li[j]):
                    c+=1
                    if j==0:
                        simu=Markov('less_5')
                        sim['state']=simu[0]
                        sim['staying_days']=simu[1]
                        sim['age_group_name']='less_5'
                        sim['country']=df.iloc[i]['country']
                    elif j==1:
                        simu=Markov('5_to_14')
                        sim['state']=simu[0]
                        sim['staying_days']=simu[1]
                        sim['age_group_name']='5_to_14'
                        sim['country']=df.iloc[i]['country']
                    elif j==2:
                        simu=Markov('15_to_24')
                        sim['state']=simu[0]
                        sim['staying_days']=simu[1]
                        sim['age_group_name']='15_to_24'
                        sim['country']=df.iloc[i]['country']
                    elif j==3:
                        simu=Markov('25_to_64')
                        sim['state']=simu[0]
                        sim['staying_days']=simu[1]
                        sim['age_group_name']='25_to_64'
                        sim['country']=df.iloc[i]['country']
                    elif j==4:
                        simu=Markov('over_65')
                        sim['state']=simu[0]
                        sim['staying_days']=simu[1]
                        sim['age_group_name']='over_65'
                        sim['country']=df.iloc[i]['country']
                    else:
                        print("Error")
                    sim['date']=dates
                    sim['id']=c
                    sim['prev_state']=simu[2]
                    rsim=rsim.append(sim,ignore_index=True)
            for a in dates:
                fr=rsim.loc[rsim['date']==a]
                d=list(fr['state'])
                su.loc[len(su.index)]=[a,df.iloc[i]['country'],d.count('D'),d.count('H'),d.count('I'),d.count('M'),d.count('S')]
            sumar=sumar.append(su)
            simulated=simulated.append(rsim,ignore_index=True)

    sumar=sumar.sort_values(by=['date','country'])
    simulated.to_csv('a3-covid-simulated-timeseries.csv',index=False)
    sumar.to_csv('a3-covid-summary-timeseries.csv',index=False)
    create_plot('a3-covid-summary-timeseries.csv',countries)


