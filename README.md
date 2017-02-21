# Python-Automated-Tools
from lxml import etree as ET1
from xml.etree import ElementTree as ET
import StringIO,string
import sys,os.path
import xlrd

######################################################################################################################
##Global Variables decleration
Range_Llmt={}
Range_Ulmt={}
Range_Llmt_Strc={}
Range_Ulmt_Strc={}
Enum_Ele1={}
Enum_Ele1_Strc={}
Consolidate_Strc={}
Consolidate={}
######################################################################################################################
## This part of code is used to parse an xml file.
def Parse_xml (filename):   
    tree= ET1.parse(filename)
    root= tree.getroot()
    NDO_Message_Name=root.xpath('//NDOMessage/@Name')
    NDO_ID=root.xpath('//NDOMessage/@ID')
    for NDO_Name in NDO_Message_Name:
        non_lsb_weight=root.xpath("//NDOMessage[@Name='"+NDO_Name+"']/DataElement[not(@LsbWeight)]/@Name")
        with_lsb_weight=root.xpath("//NDOMessage[@Name='"+NDO_Name+"']/DataElement[(@LsbWeight)]/@Name")
        Element_Ref_Name_notlsb= root.xpath("//NDOMessage[@Name='"+NDO_Name+"']/DataElement[not(@LsbWeight)]/@ElementReference")
        Element_Ref_Name_lsb=root.xpath("//NDOMessage[@Name='"+NDO_Name+"']/DataElement[(@LsbWeight)]/@ElementReference")
        data_element_names=root.xpath("//NDOMessage[@Name='"+NDO_Name+"']/DataElement/@Name")
        ##  Calling Element_Extract function
        Element_Extract(root,data_element_names,with_lsb_weight,non_lsb_weight,Element_Ref_Name_lsb,NDO_Name,Element_Ref_Name_notlsb)

##########################################################################################################################        
## To extract the Name,Datatype,ElementReference attribute from xml file.
def Element_Extract (root,data_element_names,with_lsb_weight,non_lsb_weight,Element_Ref_Name_lsb,NDO_Name,Element_Ref_Name_notlsb):
    DE_Datatype=[]
    DE_Lsbw=[]
    DE_Name=[]
    DE_Name1=[]
    DE_Name2=[]
    DE_ElementRef=[]
    lsb_count=0
    DE_Lsbw_structure=[]
    element_Reference_struture=[]
    ##This for loop is use to extract the Name,Datatype,ElementReference attribute
    for i in range(0,len(data_element_names)):
      if(data_element_names[i] in non_lsb_weight) and (data_element_names[i]!='Ndo_Revision_Number' and data_element_names[i]!='Spare'):
            DE_Name.append(root.xpath("//NDOMessage[@Name='"+NDO_Name+"']/DataElement/@Name")[i])
            DE_Datatype.append(root.xpath("//NDOMessage[@Name='"+NDO_Name+"']/DataElement/@DataType")[i])
            DE_ElementRef.append(root.xpath("//NDOMessage[@Name='"+NDO_Name+"']/DataElement/@ElementReference")[i])
      if(data_element_names[i] in with_lsb_weight) and lsb_count <= (len(root.xpath('//DataElement/@LsbWeight'))-1):
            DE_Name.append(root.xpath("//NDOMessage[@Name='"+NDO_Name+"']/DataElement[(@LsbWeight)]/@Name")[lsb_count])
            DE_ElementRef.append(root.xpath("//NDOMessage[@Name='"+NDO_Name+"']/DataElement[(@LsbWeight)]/@ElementReference")[lsb_count])
            DE_Datatype.append(root.xpath("//NDOMessage[@Name='"+NDO_Name+"']/DataElement[(@LsbWeight)]/@DataType")[lsb_count])
            lsb_count=lsb_count+1
      
    lsb_count=0
    ## This for loop is use to extract the LsbWeight attribute. 
    for i in range(0,len(data_element_names)):
        if (data_element_names[i] in non_lsb_weight) and (data_element_names[i]!='Ndo_Revision_Number') and (data_element_names[i]!='Spare'):
               DE_Lsbw.append('1')
        else:
            if (data_element_names[i]!='Ndo_Revision_Number') and (data_element_names[i]!='Spare') and lsb_count <= (len(root.xpath('//DataElement/@LsbWeight'))):
                 DE_Lsbw.append(root.xpath("//NDOMessage[@Name='"+NDO_Name+"']/DataElement/@LsbWeight")[lsb_count])
    lsb_count=0
   
    Range_Name = root.xpath('//References/Range/@Name')
    Enum_Name = root.xpath('//References/Enumeration/@Name')
    print Enum_Name
    
    for j in range(0,len(root.xpath('//References/Range/@Name'))):
        
        Range_Llmt[Range_Name[j]]=root.xpath("//References/Range[@Name='" +Range_Name[j]+"']/RangeElement/@LowerBound")
        Range_Ulmt[Range_Name[j]]=root.xpath("//References/Range[@Name='" +Range_Name[j]+"']/RangeElement/@UpperBound")
    for j in range(0,len(root.xpath('//References/Enumeration/@Name'))):
        Enum_Ele1[Enum_Name[j]]=root.xpath("//References/Enumeration[@Name='" +Enum_Name[j]+"']/EnumElement/@Name")
    
## To Generate Test data for Selected ElementReference name sequentially for non-structured datatype.
    for j in range(0,len(DE_ElementRef)):
        for i in Range_Llmt:        
             if (i == DE_ElementRef[j]) and DE_Datatype[j] == 'UnsignedInt':
                 Consolidate[DE_Name[j]]=test_data_Range_int(root,i,Element_Ref_Name_lsb,Element_Ref_Name_notlsb)
             elif (i == DE_ElementRef[j]) and DE_Datatype[j] != 'UnsignedInt' and DE_Datatype[j] != 'Char':
                    Consolidate[DE_Name[j]]=test_data_Range_float(root,i,Element_Ref_Name_lsb,Element_Ref_Name_notlsb)
             else:
                 if (i == DE_ElementRef[j]) and DE_Datatype[j] != 'Char':
                     Consolidate[DE_Name[j]]=test_data_Range_float(root,i,Element_Ref_Name_notlsb)    
        else:
            for k in Enum_Ele1:       
                if (k == DE_ElementRef[j]) :
                   Consolidate[DE_Name[j]]=test_data_Enum(k)
    for i in range(0,len(DE_Name)):
        if(DE_Datatype[i]!= 'Structure'):
            DE_Name1.append(DE_Name[i])
        else:
            DE_Name2.append(DE_Name[i])

    ## This for loop is use to extract the DataElement under substructure
    for i in range(0,len(DE_Name)):
       if(DE_Datatype[i]== 'Structure'):
           data_element_names_structure=root.xpath("//Substructure[@Name='"+DE_ElementRef[i]+"']/DataElement/@Name")
           non_lsb_weight_structure=root.xpath("//Substructure[@Name='"+DE_ElementRef[i]+"']/DataElement[not(@LsbWeight)]/@Name")
           with_lsb_weight_structure=root.xpath("//Substructure[@Name='"+DE_ElementRef[i]+"']/DataElement[(@LsbWeight)]/@Name")
           element_Reference_struture.append(DE_ElementRef[i])
    flag=1
##This for loop is use to generate test condition for struture datatype
    for i in element_Reference_struture:
        DE_Datatype_structure=[]
        DE_Name_structure=[]
        DE_ElementRef_structure =[]
        Element_Ref_lsb_Struc=[]
        Element_Ref_notlsb_Struc=[]
        if flag!=0:
##This for loop is use to extract all dataelements name,datatype,reference name under perticular substructure
            for j in range(0,len(data_element_names_structure)):
                if data_element_names_structure[j] in non_lsb_weight_structure and data_element_names_structure[j]!='Ndo_Revision_Number' and data_element_names_structure[j]!='Spare':  
                    DE_Name_structure.append(root.xpath("//Substructure[@Name='"+i+"']/DataElement/@Name")[j])
                    DE_Datatype_structure.append(root.xpath("//Substructure[@Name='"+i+"']/DataElement/@DataType")[j])
                    DE_ElementRef_structure.append(root.xpath("//Substructure[@Name='"+i+"']/DataElement/@ElementReference")[j])
                if data_element_names_structure[j] in with_lsb_weight_structure and data_element_names_structure[j]!='Ndo_Revision_Number' and data_element_names_structure[j]!='Spare':  
                  DE_Name_structure.append(root.xpath("//Substructure[@Name='"+i+"']/DataElement/@Name")[lsb_count])
                  DE_Datatype_structure.append(root.xpath("//Substructure[@Name='"+i+"']/DataElement/@DataType")[lsb_count])
                  DE_ElementRef_structure.append(root.xpath("//References/Substructure[@Name='"+i+"']/DataElement/@ElementReference")[lsb_count])
            Range_Name_Struct = root.xpath("//References/Substructure[@Name='"+i+"']/Range/@Name")
            Enum_Name_Struct = root.xpath("//References/Substructure[@Name='"+i+"']/Enumeration/@Name")
            Element_Ref_lsb_Struc= root.xpath("//References/Substructure[@Name='"+i+"']/DataElement[(@LsbWeight)]/@ElementReference")
            Element_Ref_notlsb_Struc=root.xpath("//References/Substructure[@Name='"+i+"']/DataElement[not(@LsbWeight)]/@ElementReference")
            for j in range(0,len(root.xpath("//References/Substructure[@Name='"+i+"']/Range/@Name"))):                  
                Range_Llmt_Strc[Range_Name_Struct[j]]=root.xpath("//References/Substructure/Range[@Name='" +Range_Name_Struct[j]+"']/RangeElement/@LowerBound")
                Range_Ulmt_Strc[Range_Name_Struct[j]]=root.xpath("//References/Substructure/Range[@Name='" +Range_Name_Struct[j]+"']/RangeElement/@UpperBound")
            for j in range(0,len(root.xpath("//References/Substructure[@Name='"+i+"']/Enumeration/@Name"))):
                Enum_Ele1_Strc[Enum_Name_Struct[j]]=root.xpath("//References/Substructure/Enumeration[@Name='" +Enum_Name_Struct[j]+"']/EnumElement/@Name")
              ## To Generate Test data for Selected ElementReference name sequentially for structured datatype.
            for j in range(0,len(DE_ElementRef_structure)):
                for i in Range_Llmt_Strc:        
                    if (i == DE_ElementRef_structure[j]) and DE_Datatype_structure[j] == 'UnsignedInt':
                        Consolidate_Strc[DE_Name_structure[j]]=test_data_Range_int_Structure(root,i,Element_Ref_lsb_Struc,Element_Ref_notlsb_Struc)
                        
                    elif (i == DE_ElementRef_structure[j]) and DE_Datatype_structure[j] != 'UnsignedInt' and DE_Datatype_structure[j] != 'Char':
                            Consolidate_Strc[DE_Name_structure[j]]=test_data_Range_float_Structure(root,i,Element_Ref_lsb_Struc,Element_Ref_notlsb_Struc)
                    else:
                        if (i == DE_ElementRef_structure[j]) and DE_Datatype_structure[j] == 'Char':
                            Consolidate_Strc[DE_Name_structure[j]]=test_data_Range_char_Structure(root,i,Element_Ref_notlsb_Struc)
                            
                else:
                    for k in Enum_Ele1_Strc:
                        if (k == DE_ElementRef_structure[j]) :
                            Consolidate_Strc[DE_Name_structure[j]]=test_data_Enum_Structure(k)
            flag=0 
        Consolidate.update(Consolidate_Strc)
        DE_Name1.extend(DE_Name_structure)
    print Consolidate
## Call to python code function which will generate the pyhton file for respective NDO
    Python_Code_Generation(NDO_Name,DE_Name1)
        
############################################################################################################                            
def test_data_Range_float_Structure(root,Range_Name,Element_Ref_lsb_Struc,Element_Ref_notlsb_Struc):
    Range_Llmt1=0
    Range_Ulmt1=0
    TC_Llmt=[]
    TC_Ulmt=[]
    flag=1
    for k in range(0,len(Range_Llmt_Strc[Range_Name])):
        Range_Llmt1= float(Range_Llmt_Strc[Range_Name][k])
        Range_Ulmt1= float(Range_Ulmt_Strc[Range_Name][k])
    mean=float((Range_Llmt1 + Range_Ulmt1)/2)
    b=Range_Llmt1
    c=Range_Ulmt1
    for m in Element_Ref_lsb_Struc:
        if m == Range_Name:
            lsb_value= (root.xpath("//References/Substructure/DataElement[(@ElementReference='"+m+"')]/@LsbWeight"))
            for i in range(0,4):
                if b <= mean:
                    b=float((b+mean)/2)
                    TC_Llmt.append(b)
                if c >= mean:
                    c=float((c+mean)/2)
                    TC_Ulmt.append(c)
    for k in range(0,len(Element_Ref_notlsb_Struc)):
        if(flag!=0):
            if Element_Ref_notlsb_Struc[k]== Range_Name:       
               lsb_value=1.0
               for i in range(0,4):
                   if b <= mean:
                      b=float((b+mean)/2)
                      TC_Llmt.append(b)
                   if c >= mean:
                      c=float((c+mean)/2)
                      TC_Ulmt.append(c)
               flag=0
    TC_Llmt.append(Range_Llmt1 + lsb_value)
    TC_Ulmt.append(Range_Ulmt1 - lsb_value)
    TC_Llmt.append(Range_Llmt1)
    TC_Ulmt.append(Range_Ulmt1)
    TC_Llmt.extend(TC_Ulmt)
    TC_Llmt.sort()
    return TC_Llmt
################################################################################################
def test_data_Range_int_Structure(root,Range_Name,Element_Ref_lsb_Struc,Element_Ref_notlsb_Struc):
    Range_Llmt1=0
    Range_Ulmt1=0
    TC_Llmt=[]
    TC_Ulmt=[]
    flag=1
    for k in range(0,len(Range_Llmt_Strc[Range_Name])):
        Range_Llmt1= int(float(Range_Llmt_Strc[Range_Name][k]))
        Range_Ulmt1= int(float(Range_Ulmt_Strc[Range_Name][k]))
    mean=int((Range_Llmt1 + Range_Ulmt1)/2)
    b=Range_Llmt1
    c=Range_Ulmt1

    for m in Element_Ref_lsb_Struc:
        if flag != 0:
            if m == Range_Name:
               lsb_value= (root.xpath("//References/Substructure/DataElement[(@ElementReference='"+m+"')]/@LsbWeight"))
               for i in range(0,4):
                   if b <= mean:
                      b=int((b+mean)/2)
                      TC_Llmt.append(b)
                   if c >= mean:
                      c=int((c+mean)/2)
                      TC_Ulmt.append(c)
               flag=0
    for k in range(0,len(Element_Ref_notlsb_Struc)):
        lsb_value=1
        if(flag != 0): 
            if Element_Ref_notlsb_Struc[k]== Range_Name:              
               for i in range(0,4):
                   if b <= mean:
                      b=int((b+mean)/2)
                      TC_Llmt.append(b)
                   if c >= mean:
                      c=int((c+mean)/2)
                      TC_Ulmt.append(c)
               flag=0
    TC_Llmt.append(Range_Llmt1 + lsb_value)
    TC_Ulmt.append(Range_Ulmt1 - lsb_value)
    TC_Llmt.append(Range_Llmt1)
    TC_Ulmt.append(Range_Ulmt1)
    TC_Llmt.extend(TC_Ulmt)
    TC_Llmt.sort()
    return TC_Llmt
##################################################################################################
def test_data_Range_char_Structure(root,Range_Name,Element_Ref_notlsb_Struc):
    Range_Llmt1=0
    Range_Ulmt1=0
    TC_Llmt=[]
    TC_Ulmt=[]
    flag=1
    for k in range(0,len(Range_Llmt_Strc[Range_Name])):
        Range_Llmt1= int(float(Range_Llmt_Strc[Range_Name][k]))
        Range_Ulmt1= int(float(Range_Ulmt_Strc[Range_Name][k]))
    mean=int((Range_Llmt1 + Range_Ulmt1)/2)
    b=Range_Llmt1
    c=Range_Ulmt1

    for k in range(0,len(Element_Ref_notlsb_Struc)):
        lsb_value=1
        if(flag != 0): 
            if Element_Ref_notlsb_Struc[k]== Range_Name:              
               for i in range(0,4):
                   if b <= mean:
                      b=int((b+mean)/2)
                      TC_Llmt.append(b)
                   if c >= mean:
                      c=int((c+mean)/2)
                      TC_Ulmt.append(c)
               flag=0
    TC_Llmt.append(Range_Llmt1 + lsb_value)
    TC_Ulmt.append(Range_Ulmt1 - lsb_value)
    TC_Llmt.append(Range_Llmt1)
    TC_Ulmt.append(Range_Ulmt1)
    TC_Llmt.extend(TC_Ulmt)
    TC_Llmt.sort()
    return TC_Llmt
##################################################################################################
def test_data_Enum_Structure(Enum_Name):
    Enum_Ele=[]
    Enum_Name_Struc=Enum_Name
    for k in range(0,len(Enum_Ele1_Strc[Enum_Name])):
        Enum_Ele.append(str(Enum_Ele1_Strc[Enum_Name][k]))
    return Enum_Ele

##########################################################################################
def test_data_Range_float(root,Range_Name,Element_Ref_Name_lsb,Element_Ref_Name_notlsb):
    Range_Llmt1=0
    Range_Ulmt1=0
    TC_Llmt=[]
    TC_Ulmt=[]
    flag=1
    for k in range(0,len(Range_Llmt[Range_Name])):
        Range_Llmt1= float(Range_Llmt[Range_Name][k])
        Range_Ulmt1= float(Range_Ulmt[Range_Name][k])
    mean=float((Range_Llmt1 + Range_Ulmt1)/2)
    b=Range_Llmt1
    c=Range_Ulmt1
    for m in Element_Ref_Name_lsb:
        if m == Range_Name:
            lsb_value= (root.xpath("//References/DataElement[(@ElementReference='"+m+"')]/@LsbWeight"))
            for i in range(0,4):
                if b <= mean:
                    b=float((b+mean)/2)
                    TC_Llmt.append(b)
                if c >= mean:
                    c=float((c+mean)/2)
                    TC_Ulmt.append(c)
    for k in range(0,len(Element_Ref_Name_notlsb)):
        if(flag!=0):
            if Element_Ref_Name_notlsb[k]== Range_Name:       
               lsb_value=1.0
               for i in range(0,4):
                   if b <= mean:
                      b=float((b+mean)/2)
                      TC_Llmt.append(b)
                   if c >= mean:
                      c=float((c+mean)/2)
                      TC_Ulmt.append(c)
               flag=0
    TC_Llmt.append(Range_Llmt1 + lsb_value)
    TC_Ulmt.append(Range_Ulmt1 - lsb_value)
    TC_Llmt.append(Range_Llmt1)
    TC_Ulmt.append(Range_Ulmt1)
    TC_Llmt.extend(TC_Ulmt)
    TC_Llmt.sort()
    return TC_Llmt
################################################################################################
def test_data_Range_int(root,Range_Name,Element_Ref_Name_lsb,Element_Ref_Name_notlsb):
    Range_Llmt1=0
    Range_Ulmt1=0
    TC_Llmt=[]
    TC_Ulmt=[]
    flag=1
    for k in range(0,len(Range_Llmt[Range_Name])):
        Range_Llmt1= int(float(Range_Llmt[Range_Name][k]))
        Range_Ulmt1= int(float(Range_Ulmt[Range_Name][k]))
    mean=int((Range_Llmt1 + Range_Ulmt1)/2)
    b=Range_Llmt1
    c=Range_Ulmt1
    for m in Element_Ref_Name_lsb:
        if m == Range_Name:
            lsb_value= (root.xpath("//References/DataElement[(@ElementReference='"+m+"')]/@LsbWeight"))
            for i in range(0,4):
                if b <= mean:
                    b=int((b+mean)/2)
                    TC_Llmt.append(b)
                if c >= mean:
                    c=int((c+mean)/2)
                    TC_Ulmt.append(c)
    for k in range(0,len(Element_Ref_Name_notlsb)):
        lsb_value=int(1)
        if(flag != 0): 
            if Element_Ref_Name_notlsb[k]== Range_Name:       
               for i in range(0,4):
                   if b <= mean:
                      b=int((b+mean)/2)
                      TC_Llmt.append(b)
                   if c >= mean:
                      c=int((c+mean)/2)
                      TC_Ulmt.append(c)
               flag=0
    TC_Llmt.append(Range_Llmt1 + lsb_value)
    TC_Ulmt.append(Range_Ulmt1 - lsb_value)
    TC_Llmt.append(Range_Llmt1)
    TC_Ulmt.append(Range_Ulmt1)
    TC_Llmt.extend(TC_Ulmt)
    TC_Llmt.sort()
    return TC_Llmt
##########################################################################################################
def test_data_Range_char(root,Range_Name,Element_Ref_Name_notlsb):
    Range_Llmt1=0
    Range_Ulmt1=0
    TC_Llmt=[]
    TC_Ulmt=[]
    flag=1
    for k in range(0,len(Range_Llmt[Range_Name])):
        Range_Llmt1= int(float(Range_Llmt[Range_Name][k]))
        Range_Ulmt1= int(float(Range_Ulmt[Range_Name][k]))
    mean=int((Range_Llmt1 + Range_Ulmt1)/2)
    b=Range_Llmt1
    c=Range_Ulmt1
    for k in range(0,len(Element_Ref_Name_notlsb)):
        if(flag != 0): 
            if Element_Ref_Name_notlsb[k]== Range_Name:       
               lsb_value=1
               for i in range(0,4):
                   if b <= mean:
                      b=int((b+mean)/2)
                      TC_Llmt.append(chr(b))
                   if c >= mean:
                      c=int((c+mean)/2)
                      TC_Ulmt.append(chr(c))
               flag = 0
    TC_Llmt.append(Range_Llmt1 + lsb_value)
    TC_Ulmt.append(Range_Ulmt1 - lsb_value)
    TC_Llmt.append(Range_Llmt1)
    TC_Ulmt.append(Range_Ulmt1)
    TC_Llmt.extend(TC_Ulmt)
    TC_Llmt.sort()
    return TC_Llmt
##########################################################################################################
def test_data_Enum(Enum_Name):
    Enum_Ele=[]
    Enum_Name_Struc=Enum_Name
    for k in range(0,len(Enum_Ele1[Enum_Name])):
        Enum_Ele.append(str(Enum_Ele1[Enum_Name][k]))
    return Enum_Ele

##########################################################################################################
def Python_Code_Generation(NDO_Name,DE_Name):

      fileptr=open('C:\\'+(NDO_Name+".py"),"w+")
      stringData="""
#_______________________________________________________________________________
# Rockwell_Copyright_Statement = u'\xA9 Copyright 2011  Rockwell Collins, Inc.
# All rights reserved.
#
# NAME: """+NDO_Name+""".py
#_______________________________________________________________________________

from FM_setup import *
import script
import test_log

def """+NDO_Name+ """():

# FUNCTIONAL DESCRIPTION:
#   This procedure 
#
# Ndo      = NDO_Message
# side     = CDU side test is for, 1 = left 2 = right
#
# SPECIAL INTERFACE REQUIREMENTS:
#    None.
#
# SPECIAL INITIALIZATION REQUIREMENTS:
#    None.
# 
# LIMITATIONS:
#    None.
# 
# NOTES:
#    None.
# 
#-------------------------------------------------------------------------------

# ALGORITHM AND CODE

     test_log.Start()"""+'\n'
      fileptr.write(stringData)
      for i in DE_Name:
          fileptr.write('\t'+' '+i+'_TC='+str(Consolidate[i])+'\n')            
      fileptr.write('##'+'\t'+' '+ 'DE_Name= '+ str(DE_Name)+'\n'+'\n')
      for i in DE_Name:
          fileptr.write('##'+'\t'+' '+'Verify '+i+'\n')
          fileptr.write('\t'+' '+'Verify_'+i+'('+i+'_TC)'+'\n')
          
      fileptr.write('\t'+' '+'test_log.End()'+'\n'+'# end '+ NDO_Name +'\n'+'\n')   
      for i in range(0,len(DE_Name)):
          stringData="""
# .............................................................................."""+'\n'
          fileptr.write(stringData)
          fileptr.write('def Verify_'+DE_Name[i]+'('+DE_Name[i]+'_TC,'+'Cdu = CDU1):'+'\n')
          stringData1="""
#-------------------------------------------------------------------------------
# FUNCTIONAL DESCRIPTION:
#   This procedure 
#
# Ndo      = def Verify_"""+DE_Name[i]+"""("""+DE_Name[i]+"""_TC,Cdu = CDU1)
# side     = CDU side test is for, 1 = left 2 = right
#
# SPECIAL INTERFACE REQUIREMENTS:
#    None.
#
# SPECIAL INITIALIZATION REQUIREMENTS:
#    None.
# 
# LIMITATIONS:
#    None.
# 
# NOTES:
#    None.
# 
#-------------------------------------------------------------------------------

    # ALGORITHM AND CODE"""+'\n'
          fileptr.write(stringData1)
          stringData2="""     for test in """+DE_Name[i]+"""_TC:
                Cdu."""
          fileptr.write(stringData2)
          Mapping_xls=xlrd.open_workbook('C:\\Python_Code\\DE_mapping.xls')
          sh = Mapping_xls.sheet_by_index(0)
          xls_DE_Names=sh.col_values(1)
          xls_CDU_page_Names=sh.col_values(2)
          xls_Field_Names=sh.col_values(3)
          for j in range(0,len(xls_DE_Names)):
              if(DE_Name[i]==xls_DE_Names[j]):
                stringData3=""""""+xls_CDU_page_Names[j]+"""."""+xls_Field_Names[j]+""""""
                fileptr.write(stringData3)            
          stringData4=""" = """+DE_Name[i]+"""_TC[test]
                script.Delay(0.5)
                FM.Verify_Element("""+'"'+DE_Name[i]+"""","""+DE_Name[i]+"""_TC[test])
# end Verify_"""+DE_Name[i]+""" """ + '\n'
          fileptr.write(stringData4)
      fileptr.close()
##############################################################################################################################
