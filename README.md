# Project_Dyno
# ---------- The base file to get searching base as endaksa^^sehır,il,ilce^^---------##

endeksa = pd.read_csv("endeksa.csv")

##-------------This file will not be edited---------##
###--------------- FOR THE DATA COLLECTION MANUALLY ONLY FROM THE INTERNET (GOOGLE.API)----------------------##
###---------------FUNCTION WILL ONLY GET SEARCHED ITEMS ACCORDING TO THE ENDEKSA ORDER.
def data_collection_manual(searched_item,how_many_item):
    total = []
    step1 = gmaps.places(query = searched_item)
    step2 = json.dumps(step1)
    step3 = json.loads(step2)
    step4 = pd.DataFrame(step3["results"])
    total.append(step4.iloc[:how_many_item])
    step5 = pd.concat(total)
    return step5


#  THE FUNCTION THAT COLLECTS EVERY PLACES THAT IS SEARCHED FROM THE ORDER IN THE "ENDEKSA" CSV FILE##

def data_collection_total(searching_key,how_many_results):
    total = []
    for i in range(0,len(endeksa)):
        
        search_loction = gmaps.places(query = endeksa["İL"][i] + " " + endeksa["İLÇE"][i] + " " + endeksa["MAHALLE"][i]+ " " + searching_key)
        file = json.dumps(search_loction)
        file_new = json.loads(file)
        df = pd.DataFrame(file_new["results"])
        total.append(df.iloc[:how_many_results])
        saved_file_name_beta = pd.concat(total)
    return saved_file_name_beta

##  SPLIT JSON LONGITUDE ALTITUDE VALUES INTO df AND REMOVE UNNECCESARY COLUMNS FOR ML ##

def api_data_adjustment(df):

    df["a"] = df["geometry"].astype(str).str.extract(':(.{,45})')
    df["b"] = df["a"].str.split(',').str[0]
    df["Latitude"] = df["b"].str[8:]
    df.drop(columns =["a","b"],axis = 1,inplace = True)
    df["c"] = df["geometry"].astype(str).str.split(",").str[1]
    df["d"] = df["c"].astype(str).str[7:]
    df["Longitude"] = df["d"].astype(str).str[:-1]
    df.drop(columns=["c","d"],axis=1,inplace=True)
    df.drop(columns=["geometry"],axis=1,inplace = True)
    df = df[["name","formatted_address","Latitude","Longitude"]]
    df.drop_duplicates(inplace = True)
    df.reset_index(drop=True, inplace=True)
    df["Latitude"] = pd.to_numeric(df["Latitude"])
    df["Longitude"] = pd.to_numeric(df["Longitude"])
    df["name"] = df["name"].astype(str)
    return df

### -----REMOVE UNNECESSARY VALUES THAT COMES FROM API SEARCHING(FALSE SEARCED RESULTS)--------##


def remove_unrelated(df,searched_word1,searched_word2):
    for i in range(0,len(df)):
        if searched_word1 not in df["name"][i] and searched_word2 not in df["name"][i] :
            df.drop([i], axis=0, inplace=True)
    return df
#### --- It will clean the same results from different dataframes (dynobil_oppoent df will have dynobil locations by accident.They should be removed )---###

def same_loc_cleaner(changed_df,according_to):
     
     changed_df["marked"] = 0
     for j in range(0,len(changed_df)):
        for i in range(0,len(according_to)):
            if float(according_to["Longitude"][i])== float(changed_df["Longitude"][j]):
                changed_df["marked"][j] =  1
                
           
  
     for i in range(0,len(changed_df)):
        if changed_df["marked"][i] == 1:
            changed_df.drop([i],axis = 0 , inplace = True)
     changed_df.drop(columns =["marked"],axis = 0,inplace = True)
     
     return changed_df
### ----Function will decide closest location from parameters---#####

def closest_calculation(origin,df_closest):
    rak = []
    rak_min = []
    if str(origin) != str(df_closest):

        for i in range (0,len(origin)):
            for j in range(0,len(df_closest)):
                rak.append(geopy.distance.geodesic((origin["Latitude"][i],origin["Longitude"][i]),(df_closest["Latitude"][j],df_closest["Longitude"][j])).km)
        
            rak_min.append(min(rak))
            rak = []
        return rak_min
    else:
        dyn = []
        dyn_min = [] 
        for i in range (0,len(origin)):
            for j in range(0,len(origin)):
                dyn.append(geopy.distance.geodesic((origin["Latitude"][i],origin["Longitude"][i]),(origin["Latitude"][j],origin["Longitude"][j])).km)
            dyn.sort()
            dyn_min.append(dyn[1])    
            dyn = [] 
        return dyn_min

import collections

def which_are_duplicated(df):
    df["duplicated"] = df.duplicated()
    a = []
    for i in range(0,len(df)):
        if df["duplicated"][i] == True:
            a.append(df.iloc[i])
    return a

dyno_data = data_collection_total("dynobil",2)

alpha = api_data_adjustment(dyno_data)
## ------------------------------------------- MANUAL CLEANING----------------------------------------####
## -------first phase-----#######
index_numbers = [2,3,5,8,10,12,16,23,27,28,35,36,37,40,42,47,57,61,
63,66,68,70,72,74,83,85,84,93,98,103,105,107,109,111,113,114,121,123,
125,128,134,139,140,150,152,153,156,160,161,163,165,167,171,175,181,
189,191,195,196,199,205,207,212,218,219,223,224,225,230,234,237,241,244,254,257,258]
beta = alpha.drop(index_numbers, axis=0)
beta.reset_index(drop=True, inplace=True)
beta
### second phase#####
new_indexes = [11,13,64,22,32,97,123,72,53,62,90,83,104,150,182,178]
theta = beta.drop(new_indexes, axis=0)
theta.reset_index(drop=True, inplace=True)
theta
### getting missing location manually####
a1 = data_collection_manual("adana ceyhan ulus dynobil",1)
a2 = data_collection_manual("ankara çankaya birlik dynobil",1)
a4 = data_collection_manual("ankara etimesgut şaşmaz dynobil",1)
a5 = data_collection_manual("ankara keçiören havacık dynobil",1)
a8 = data_collection_manual("bursa osmangazi demirtaşpaşa dynobil",1) 
a9 = data_collection_manual("bursa osmangazi emekzekai gümüşdiş dynobil",1) 
a10 = data_collection_manual("denizli acıpayam dynobil",1) 
a15 = data_collection_manual("karabük dynobil",1) 
a19 = data_collection_manual("mersin silifke dynobil",1) 


total = [a1,a2,a4,a5,a8,a9,a10,a15,a19]
collected_data = api_data_adjustment(pd.concat(total))
#### ---THE REASON WHY THE CODE WAS NOT WORKING PROPERLY IS THAT SOME LOCATIONS IN ENDEKSA CSV FILE DOES NOT EXIST!!!!!!!!-------#######
#### ----- And the ones that were missing are the locations that their names do not look like a dynobil locations-----##### (HUMAN ERRORS)
collected_data

theta.loc[1.5] = collected_data.loc[0]
theta.loc[18.5] = collected_data.loc[1]
theta.loc[22.5] = collected_data.loc[2]
theta.loc[25.5] = collected_data.loc[3]
theta.loc[68.5] = collected_data.loc[4]
theta.loc[70.5] = collected_data.loc[5]
theta.loc[76.5] = collected_data.loc[6]
theta.loc[118.5] = collected_data.loc[7]
theta.loc[142.5] = collected_data.loc[8]
theta = theta.sort_index().reset_index(drop=True)
theta
##### ALL THE DYNOBIL LOCATIONS ARE SYCNED WITH ENDEKSA VALUES########
dynobil_locations = theta
#### data collection for dynobil opponents######
first_part = data_collection_total("oto ekspertiz",20)
second_part = api_data_adjustment(first_part)
third_part = remove_unrelated(second_part,"ksper","KSPER")
third_part = third_part.sort_index().reset_index(drop=True)
third_part
dynobil_oponents = same_loc_cleaner(third_part,dynobil_locations)
dynobil_oponents
#### data collection for NOTER######
first_part1 = data_collection_total("noter",20)
second_part2 = api_data_adjustment(first_part1)
third_part3 = remove_unrelated(second_part2,"oter","OTER")
third_part3 = third_part3.sort_index().reset_index(drop=True)
Noter = same_loc_cleaner(third_part3,dynobil_locations)
#### data collection for GALERI######
first_part11 = data_collection_total("oto galeri",20)
second_part22 = api_data_adjustment(first_part11)
third_part33 = remove_unrelated(second_part22,"aler","ALER")
third_part33 = third_part33.sort_index().reset_index(drop=True)
Galeri = same_loc_cleaner(third_part33,dynobil_locations)
Galeri
### ----- CALCULATING CLOSEST DYNOBIL LOCATIONS-----###


closest_dynobil = closest_calculation(dynobil_locations,dynobil_locations)
closest_oponent = closest_calculation(dynobil_locations,dynobil_oponents)
closest_noter = closest_calculation(dynobil_locations,Noter)
closest_galeri = closest_calculation(dynobil_locations,Galeri)
### EDITING THE ENDEKSA VALUES FOR ML ALGORITHM###
unedited_endeksa = pd.read_csv("endeksa.csv")

unedited_endeksa
unedited_endeksa = unedited_endeksa.drop(columns=["İL","İLÇE","MAHALLE"])
unedited_endeksa.dtypes
unedited_endeksa.isnull().sum()
unedited_endeksa["GAYRİMENKUL_İŞLEMLERİ"].fillna(unedited_endeksa["GAYRİMENKUL_İŞLEMLERİ"].median(skipna=True), inplace=True)

unedited_endeksa["GAYRİMENKUL_DAĞILIMI"].fillna(unedited_endeksa["GAYRİMENKUL_DAĞILIMI"].median(skipna=True), inplace=True)
unedited_endeksa.isnull().sum()
unedited_endeksa["TÜKETİM_EĞİLİMLERİ"].fillna(unedited_endeksa["TÜKETİM_EĞİLİMLERİ"].median(skipna=True), inplace=True)
unedited_endeksa["HANE_HALKI"].fillna(unedited_endeksa["HANE_HALKI"].median(skipna=True), inplace=True)
unedited_endeksa["NÜFUS"].fillna(unedited_endeksa["NÜFUS"].median(skipna=True), inplace=True)
unedited_endeksa["NÜFUS_YOĞ."].fillna(unedited_endeksa["NÜFUS_YOĞ."].median(skipna=True), inplace=True)
unedited_endeksa.isnull().sum()
import matplotlib as plt

ax = unedited_endeksa['EĞİTİM_DÜZEYİ'].hist(bins=15, density=True, stacked=True, alpha=0.7)
ax.set(xlabel='Age')

ax = unedited_endeksa['SOSYO_EKONOMİK_STATÜ'].hist(bins=15, density=True, stacked=True, alpha=0.7)
ax.set(xlabel='Age')
unedited_endeksa['EĞİTİM_DÜZEYİ'].fillna("LİSE",inplace=True)
unedited_endeksa['SOSYO_EKONOMİK_STATÜ'].fillna("C",inplace=True)
unedited_endeksa
unedited_endeksa = pd.get_dummies(unedited_endeksa, columns=["EĞİTİM_DÜZEYİ","SOSYO_EKONOMİK_STATÜ"])
main = dynobil_locations
main["closest_dynobil"] = closest_dynobil
main["closest_oponent"] = closest_oponent
main["closest_noter"] = closest_noter
main["closest_galeri"] = closest_galeri

main = pd.concat([main,unedited_endeksa],axis =1)
#### ---I DO NOT WANT TO USE API REQUEST SO I JUST UPLOAD FROM CSV-----#######

main = pd.read_csv("main.csv")
main = main.iloc[:, :-2]
main
main = pd.get_dummies(main, columns=["EĞİTİM_DÜZEYİ","SOSYO_EKONOMİK_STATÜ"])
main.to_csv("main.csv")