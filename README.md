# Compression - temperature/time data
import numpy as np
import math
import matplotlib.pyplot as plt
import datetime
import random
plt.rcParams.update({'font.size': 16})
##########################################################################################

# Set universal constants
kappa = 1.380649*10**(-23)
planck = 6.62607015*10**(-34)
R = 8.31446261815324
number_species,temperatures_list,times_list,quantity_list = [],[],[],[]

for num_loop in range(-70,140,2):

	np.random.seed(10)
	
##########################################################################################
#################################### THINGS TO CHANGE ####################################
##########################################################################################
	num = 12
	feeding_0,feeding_1 = 0,1
	plotting = 0
	precision_dt = 1.#20
	threshold_detection = 0.00001
	
	# SET LOOP CONDITION
	#T=40.+272
	T=num_loop*1.+272
	time_end = 10000#+num_loop
	#time_end = 1.*np.exp(0.1*num_loop) 
	IC = 0.05
	#IC = .01*(num_loop+5)

	# SET RANDOMNESS
	randomnes_H = .5#.8 #SI =0.1,0.2,0.3,0.5,0.8
	randomnes_Ea = .5#.8

	# SET INITIAL CONDITIONS
	M,M0 = np.full(shape=num, fill_value=0.05),np.full(shape=num, fill_value=0.05)
	P2,P20 = np.full(shape=(num,num),fill_value=0.0000001),np.zeros((num,num))
	P3,P30 = np.full(shape=(num,num,num),fill_value=0.0000001),np.zeros((num,num,num))
	P4,P40 = np.full(shape=(num,num,num,num),fill_value=0.0000001),np.zeros((num,num,num,num))
	P5,P50 = np.full(shape=(num,num,num,num,num),fill_value=0.0000001),np.zeros((num,num,num,num,num))
	W,W0 = 1.,1.
	M0[0]=IC
	M0[1]=IC

##########################################################################################
################################### CHANGING DOMINANCE ###################################
##########################################################################################

	# Defining Monomers
	HM,SM,GM = np.zeros(num),np.zeros(num),np.zeros(num)
	for i in range(num):
		HM[i] = 5.00*4.184*1000.
		SM[i] = 10.00*4.184*1000./300
		GM[i] = HM[i]-T*SM[i]
	# Defining Dimers
	HP2,SP2,GP2 =  np.zeros((num,num)),np.zeros((num,num)),np.zeros((num,num))
	for i in range(num):
		for j in range(num):
	################################ Thermodynamic dominance #############################
			if i==0 or j==0:
				HP2[i][j] = HM[i]+HM[j] + 8.00*4.184*1000.+np.random.uniform(-randomnes_H,randomnes_H,1)*4.184*1000.
			else:
				HP2[i][j] = HM[i]+HM[j] + 8.00*4.184*1000.+np.random.uniform(-randomnes_H,randomnes_H,1)*4.184*1000.
	##########################################################################################
			SP2[i][j] = SM[i]+SM[j] + 10.00*4.184*1000./300
			GP2[i][j] = HP2[i][j]-T*SP2[i][j]

	#Defining thermodynamics M to P2
	H2,S2,G2,G2_act,Ea2_mp,Ea2_pm,k2_mp,k2_pm = np.zeros((num,num)),np.zeros((num,num)),np.zeros((num,num)),np.zeros((num,num)),np.zeros((num,num)),np.zeros((num,num)),np.zeros((num,num)),np.zeros((num,num))		
	for i in range(num):
		for j in range(num):
			H2[i][j] = HP2[i][j]-HM[i]-HM[j]
			S2[i][j] = SP2[i][j]-SM[i]-SM[j]
			G2[i][j] = GP2[i][j]-GM[i]-GM[j]
	#################################### Kinetic dominance ###############################
			if i==1 or j==1:
				G2_act[i][j] = 21.*4.184*1000.+ 1.*np.random.uniform(-randomnes_Ea,randomnes_Ea,1)*4.184*1000.
			else:
				G2_act[i][j] = 26.*4.184*1000.+ 1.*np.random.uniform(-randomnes_Ea,randomnes_Ea,1)*4.184*1000.
	##########################################################################################		
	#Defining kinetics M to P2
			if G2[i][j]<0.:
				Ea2_mp[i][j] = G2_act[i][j]
				Ea2_pm[i][j] = np.abs(G2[i][j]) + G2_act[i][j]
			else:
				Ea2_mp[i][j] = G2[i][j] + G2_act[i][j]
				Ea2_pm[i][j] = G2_act[i][j]

##########################################################################################
################################ CHANGING CONNECTIVITY ###################################
##########################################################################################
			
############### CONNECTIVITY ALL	
			k2_mp[i][j] = (kappa*T/planck)*np.exp(-(Ea2_mp[i][j])/(R*T))
			k2_pm[i][j] = (kappa*T/planck)*np.exp(-(Ea2_pm[i][j])/(R*T))
############### CONNECTIVITY 3
# 			if ((i==0 or i==1 or i==2) and (j==0 or j==1 or j==2)):
# 				lll+=1
# 				k2_mp[i][j] = (kappa*T/planck)*np.exp(-(Ea2_mp[i][j])/(R*T))
# 				k2_pm[i][j] = (kappa*T/planck)*np.exp(-(Ea2_pm[i][j])/(R*T))
# 			elif ((i==3 or i==4 or i==5) and (j==3 or j==4 or j==5)):
# 				lll+=1
# 				k2_mp[i][j] = (kappa*T/planck)*np.exp(-(Ea2_mp[i][j])/(R*T))
# 				k2_pm[i][j] = (kappa*T/planck)*np.exp(-(Ea2_pm[i][j])/(R*T))
# 			elif ((i==6 or i==7 or i==8) and (j==6 or j==7 or j==8)):
# 				lll+=1
# 				k2_mp[i][j] = (kappa*T/planck)*np.exp(-(Ea2_mp[i][j])/(R*T))
# 				k2_pm[i][j] = (kappa*T/planck)*np.exp(-(Ea2_pm[i][j])/(R*T))
# 			elif ((i==9 or i==10 or i==11) and (j==9 or j==10 or j==11)):
# 				lll+=1
# 				k2_mp[i][j] = (kappa*T/planck)*np.exp(-(Ea2_mp[i][j])/(R*T))
# 				k2_pm[i][j] = (kappa*T/planck)*np.exp(-(Ea2_pm[i][j])/(R*T))
# 			else:
# 				k2_mp[i][j] = 0.
# 				k2_pm[i][j] = 0.
############## CONNECTIVITY 4
# 			if ((i==0 or i==1 or i==2 or i==3) and (j==0 or j==1 or j==2 or j==3)):
# 				lll+=1
# 				k2_mp[i][j] = (kappa*T/planck)*np.exp(-(Ea2_mp[i][j])/(R*T))
# 				k2_pm[i][j] = (kappa*T/planck)*np.exp(-(Ea2_pm[i][j])/(R*T))
# 			elif ((i==4 or i==5 or i==6 or i==7) and (j==4 or j==5 or j==6 or j==7)):
# 				lll+=1
# 				k2_mp[i][j] = (kappa*T/planck)*np.exp(-(Ea2_mp[i][j])/(R*T))
# 				k2_pm[i][j] = (kappa*T/planck)*np.exp(-(Ea2_pm[i][j])/(R*T))
# 			elif ((i==8 or i==9 or i==10 or i==11) and (j==8 or j==9 or j==10 or j==11)):
# 				lll+=1
# 				k2_mp[i][j] = (kappa*T/planck)*np.exp(-(Ea2_mp[i][j])/(R*T))
# 				k2_pm[i][j] = (kappa*T/planck)*np.exp(-(Ea2_pm[i][j])/(R*T))
# 			else:
# 				k2_mp[i][j] = 0.
# 				k2_pm[i][j] = 0.
############### CONNECTIVITY 6
# 			if ((i==0 or i==1 or i==2 or i==3 or i==4 or i==5) and (j==0 or j==1 or j==2 or j==3 or j==4 or j==5)):
# 				#lll+=1
# 				k2_mp[i][j] = (kappa*T/planck)*np.exp(-(Ea2_mp[i][j])/(R*T))
# 				k2_pm[i][j] = (kappa*T/planck)*np.exp(-(Ea2_pm[i][j])/(R*T))
# 			elif ((i==6 or i==7 or i==8 or i==9 or i==10 or i==11) and (j==6 or j==7 or j==8 or j==9 or j==10 or j==11)):
# 				#lll+=1
# 				k2_mp[i][j] = (kappa*T/planck)*np.exp(-(Ea2_mp[i][j])/(R*T))
# 				k2_pm[i][j] = (kappa*T/planck)*np.exp(-(Ea2_pm[i][j])/(R*T))
# 			else:
# 				k2_mp[i][j] = 0.
# 				k2_pm[i][j] = 0.

##########################################################################################
#################################### SETTING 'dt' ########################################
##########################################################################################
	max_k = max(k2_mp.max(),k2_pm.max())
	dt = precision_dt/max_k
	time = int(time_end/dt)
	if time<100:
		dt = time_end/100.
		time = int(time_end/dt)
	if time > 5000000:
		time = 111
	print ('K*dt',dt*max_k)
	print ('dt(s)',dt)
	print ('Time steps',"{:e}".format(time))
	print ('R.T.(s)',"{:e}".format(time_end))
	print ('Temperature:',T-272)
	print ('Quantity:',IC)

	# Initialazing plot
	M_plot,P2_plot,W_plot,time_plot = [],[],[],[]
	for i in range(num):
		M_plot.append([])
		M_plot[i].append(M[i])
		P2_plot.append([])
		for j in range(num):
			P2_plot[i].append([])
			P2_plot[i][j].append(P2[i][j])
	W_plot.append(W)
	time_plot.append(0.000001*dt)

##########################################################################################
################################## RUNNING SIMULATION ####################################
##########################################################################################

	for t in range(time):
		if (100*t/time)%10==0:
			print (100*t/time)
		W=0.5
		for l in range(num):
			increase_M_2,decrease_M_2 = 0.,0.
			for i in range(num):
				for j in range(num):
					if i ==l or j == l:
						if i==l and j == l:
							increase_M_2 += +2.*k2_pm[i][j]*P20[i][j]*W0
							decrease_M_2 += -2.*k2_mp[i][j]*M0[i]*M0[j]
						else:
							increase_M_2 += +k2_pm[i][j]*P20[i][j]*W0
							decrease_M_2 += -k2_mp[i][j]*M0[i]*M0[j]
					if l==0:
						P2[i][j] = dt*(k2_mp[i][j]*M0[i]*M0[j]-k2_pm[i][j]*P20[i][j]*W0) + P20[i][j]

			if (feeding_0==1 or feeding_1==1):						
				if (l==0 and feeding_0==1):
					M[l] = 0.05
				elif (l==1 and feeding_1==1):
					M[l] = 0.05
				else:
					M[l] = dt*(increase_M_2+decrease_M_2)+M0[l]
			else:
				M[l] = dt*(increase_M_2+decrease_M_2)+M0[l]
			
		M0,P20,P30,P40,P50,W0 = M,P2,P3,P4,P5,W
			
##########################################################################################
################################# SAVE DATA FOR PLOT #####################################
##########################################################################################
		if plotting == 1:
			if t<100:	
				W_plot.append(W)
				time_plot.append((t+1)*dt)#/3600.)
				for i in range(num):
					M_plot[i].append(M[i])
					for j in range(num):
						P2_plot[i][j].append(P2[i][j])
			else:
				if t%100==0:
					W_plot.append(W)
					time_plot.append((t+1)*dt)#/3600.)
					for i in range(num):
						M_plot[i].append(M[i])
						for j in range(num):
							P2_plot[i][j].append(P2[i][j])
		else:
			pass

##########################################################################################
############################ NUMBER SPECIES DETECTOR #####################################
##########################################################################################

	non_zero = 0
	for i in range(num):
		if M0[i]> threshold_detection:
			non_zero +=1
		for j in range(num):
			if P20[i][j]>threshold_detection:
				non_zero +=1 
	
	number_species.append(non_zero)
	temperatures_list.append(T)
	times_list.append(time_end)
	quantity_list.append(IC)
	
	print ('Species',number_species)
	print ('Tempera',temperatures_list)
	print ('Time',times_list)
	print ('Quantity',quantity_list)

##########################################################################################
################################# MAKING PLOTS ###########################################
##########################################################################################

for i in range(num):
	if i==1 or i==0:
	#if i==1:
		transparent=1.
	else:
		transparent=.2
	plt.plot(time_plot,M_plot[i],c='r',alpha=transparent)
	for j in range(num):
		#plt.plot(time_plot,P2_plot[i][j],c='b',alpha=1.-float(i+1)/num)
		plt.plot(time_plot,P2_plot[i][j],c='b',alpha=transparent)

plt.xscale('log',base=10)
plt.yscale('log',base=10)
#plt.ylim(0.000001, 10.00)
#plt.xlim(20000, 20000000000)#0.85
#plt.xlim(2, 200000)#0.85
plt.title('T='+str(T)+' time='+str(time_end)+' Q='+str(IC))
plt.axhline(y=threshold_detection, color='gray', linestyle='--',linewidth=1.)
plt.axvline(x=10000, color='gray', linestyle='--',linewidth=1.)
#plt.scatter(10000000000,0.00001,color='gray')
plt.savefig(str(datetime.datetime.now)+'.svg', dpi=300)
plt.show()


