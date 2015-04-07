# Python
__author__ = 'Matt Russo'

#    Parameters:
#    0: Output Workspace
#    1: Year(String) Ex. 2015
#    2: Input Feature Class
#    3: Clip Feature
#    4: Join Table (DMG Type)
#    5: Join Table (Host Type)
#    6: Join Table (Forest Type)
#    7: Erase Feature


import arcpy
import log
from arcpy import env

log.header('Pine Beetle Script')

# Environment Settings
env.overwriteOutput = True
env.addOutputsToMap = True
env.qualifiedFieldNames = False
env.workspace = arcpy.GetParameterAsText(0)  # Parameter 0

# Year Information
yearString = arcpy.GetParameterAsText(1)  # Parameter 1
year = str(yearString)

# Outputs
r = 'r'
clipOutput = r + yearString + 'C'
joinOutput = r + yearString + 'J'
queryOutput = r + yearString + 'Q'
eraseOutput = r + yearString + 'E'

# Layer Files
layerOutput1 = 'joinLayer'
layerOutput2 = 'queryLayer'


def clip():
    """ Clip Analysis
    """
    clipInput, clipFeature = arcpy.GetParameterAsText(2), arcpy.GetParameterAsText(3)  # Parameters 2, 3
    log.message('Clipping ' + clipInput + ' to ' + clipFeature)
    arcpy.Clip_analysis(clipInput, clipFeature, clipOutput)
    log.message(clipOutput + ' Created')


clip()


def join():
    """ Join USDA Table
    """
    joinTable1, joinTable2, joinTable3 = arcpy.GetParameterAsText(4), \
                                         arcpy.GetParameterAsText(5), \
                                         arcpy.GetParameterAsText(6)  # Parameters 4, 5, 6
    inputField1, inputField2, inputField3 = 'DCA1', 'HOST1', 'FOR_TYPE1'
    joinField = 'CODE'

    log.message('Creating ' + layerOutput1 + ' to join the tables')
    arcpy.MakeFeatureLayer_management(clipOutput, layerOutput1)[0]
    log.message('Joining the tables ...')
    arcpy.AddJoin_management(layerOutput1, inputField1, joinTable1, joinField)
    log.message('Joined: ' + joinTable1)
    arcpy.AddJoin_management(layerOutput1, inputField2, joinTable2, joinField)
    log.message('Joined: ' + joinTable2)
    arcpy.AddJoin_management(layerOutput1, inputField3, joinTable3, joinField)
    log.message('Joined: ' + joinTable3)
    log.message('Creating ' + joinOutput + ' from ' + layerOutput1)
    arcpy.CopyFeatures_management(layerOutput1, joinOutput)
    log.message(joinOutput + ' Created')


join()


def query():
    """ Master Query
    """
    y = 'Year'
    t = 'Text'
    c = 'CLEAR_SELECTION'
    n = 'NEW_SELECTION'
    q = '!Common_Nam!'
    py = 'PYTHON_9.3'
    nr = 'NON_REQUIRED'
    dmg = 'DMGTYPE'
    mpb = 'MPB'
    null = 'NULLABLE'

    log.message('Adding Fields to ' + joinOutput)
    arcpy.AddField_management(joinOutput, y, t, 10, '', 100, '', null, nr)
    log.message(y + ' has been added')
    arcpy.AddField_management(joinOutput, dmg, t, 10, '', 100, '', null, nr)
    log.message(dmg + ' has been added')
    arcpy.AddField_management(joinOutput, mpb, t, 10, '', 100, '', null, nr)
    log.message(mpb + ' has been added')

    log.message('Creating ' + layerOutput2 + ' to run the query')
    arcpy.MakeFeatureLayer_management(joinOutput, layerOutput2)[0]
    arcpy.CalculateField_management(layerOutput2, y, year, py)
    arcpy.CalculateField_management(layerOutput2, dmg, q, py)
    log.message('Query ...')

    # Aspen Decline
    where_clause = """
        ("DCA1" = 80001)
        OR ("DCA2" = 80001)
        OR ("DCA3" = 80001)
    """
    arcpy.SelectLayerByAttribute_management(layerOutput2, n, where_clause)
    arcpy.CalculateField_management(layerOutput2, dmg, 'str("Aspen Decline")', py, "")
    arcpy.SelectLayerByAttribute_management(layerOutput2, c)
    log.message('Aspen Delcine Complete')

    # MPB - Mountain Pine Beetle
    where_clause = """
        "DCA1" IN(80003, 11006)
        OR "DCA2" IN(80003, 11006)
        OR "DCA3" IN(80003, 11006)
    """
    arcpy.SelectLayerByAttribute_management(layerOutput2, n, where_clause)
    arcpy.CalculateField_management(layerOutput2, mpb, 'str("True")', py, "")
    arcpy.SelectLayerByAttribute_management(layerOutput2, c)
    log.message('Mountain Pine Beetle Complete')

    # LP - Lodge Pole
    where_clause = """
        ("DCA1" = 11006 AND "HOST1" = 108)
        OR ("DCA2" = 11006 AND "HOST2" = 108)
        OR ("DCA3" = 11006 AND "HOST3" = 108)
    """
    arcpy.SelectLayerByAttribute_management(layerOutput2, n, where_clause)
    arcpy.CalculateField_management(layerOutput2, dmg, 'str("Lodge Pole Pine")', py, "")
    arcpy.SelectLayerByAttribute_management(layerOutput2, c)
    log.message('Lodge Pole Pine Complete')

    # PP - Ponderosa Pine
    where_clause = """
        ("DCA1" = 11006 AND "HOST1" = 122)
        OR ("DCA2" = 11006 AND "HOST2" = 122)
        OR ("DCA3" = 11006 AND "HOST3" = 122)
    """
    arcpy.SelectLayerByAttribute_management(layerOutput2, n, where_clause)
    arcpy.CalculateField_management(layerOutput2, dmg, 'str("Ponderosa Pine")', py, "")
    arcpy.SelectLayerByAttribute_management(layerOutput2, c)
    log.message('Ponderosa Pine Complete')

    # 5N - Limber Pine
    where_clause = """
        ("DCA1" = 80003)
        OR ("DCA2" = 80003)
        OR ("DCA3" = 80003)
    """
    arcpy.SelectLayerByAttribute_management(layerOutput2, n, where_clause)
    arcpy.CalculateField_management(layerOutput2, dmg, 'str("Limber Pine")', py, "")
    arcpy.SelectLayerByAttribute_management(layerOutput2, c)
    log.message('Limber Pine Complete')

    # SB - Spruce Beetle
    where_clause = """
        ("DCA1" = 11009)
        OR ("DCA2" = 11009)
        OR ("DCA3" = 11009)
    """
    arcpy.SelectLayerByAttribute_management(layerOutput2, n, where_clause)
    arcpy.CalculateField_management(layerOutput2, dmg, 'str("Spruce Beetle")', py, "")
    arcpy.SelectLayerByAttribute_management(layerOutput2, c)
    log.message('Spruce Beetle Complete')

    # DFB - Douglas-Fir Beetle
    where_clause = """
        ("DCA1" = 11007)
        OR ("DCA2" = 11007)
        OR ("DCA3" = 11007)
    """
    arcpy.SelectLayerByAttribute_management(layerOutput2, n, where_clause)
    arcpy.CalculateField_management(layerOutput2, dmg, 'str("Douglas Fir Beetle")', py, "")
    arcpy.SelectLayerByAttribute_management(layerOutput2, c)
    log.message('Douglas Fir Beetle Complete')

    # WBBB - Sub-Alpine Fir Mortality
    where_clause = """
        ("DCA1" = 80002)
        OR ("DCA2" = 80002)
        OR ("DCA3" = 80002)
    """
    arcpy.SelectLayerByAttribute_management(layerOutput2, n, where_clause)
    arcpy.CalculateField_management(layerOutput2, dmg, 'str("Sub-Alpine Fir Mortality")', py, "")
    arcpy.SelectLayerByAttribute_management(layerOutput2, c)
    log.message('Sub-Alpine Fir Mortality')

    # WSB - Western Spruce Budworm
    where_clause = """
        ("DCA1" = 12040)
        OR ("DCA2" = 12040)
        OR ("DCA3" = 12040)
    """
    arcpy.SelectLayerByAttribute_management(layerOutput2, n, where_clause)
    arcpy.CalculateField_management(layerOutput2, dmg, 'str("Western Spruce Budworm")', py, "")
    arcpy.SelectLayerByAttribute_management(layerOutput2, c)
    log.message('Western Spruce Budworm')

    # WPB - Western Pine Beetle
    where_clause = """
        ("DCA1" = 11002)
        OR ("DCA2" = 11002)
        OR ("DCA3" = 11002)
    """
    arcpy.SelectLayerByAttribute_management(layerOutput2, n, where_clause)
    arcpy.CalculateField_management(layerOutput2, dmg, 'str("Western Pine Beetle")', py, "")
    arcpy.SelectLayerByAttribute_management(layerOutput2, c)
    log.message('Western Pine Beetle')

    log.message('Creating ' + queryOutput + ' from ' + layerOutput2)
    arcpy.CopyFeatures_management(layerOutput2, queryOutput)
    log.message(queryOutput + ' Created')


query()


def erase():
    """Erase Analysis
"""
eraseFeature = arcpy.GetParameterAsText(7)  # Parameter 7
log.message('Erasing ' + queryOutput + ' to ' + eraseFeature)
arcpy.Erase_analysis(queryOutput, eraseFeature, eraseOutput)

erase()


def delfields():
    """ Deletes Fields in List
    """
    fields = arcpy.GetParameterAsText(8)  # Parameter 8
    fieldList = fields.split(";")
    fieldList.sort()

    try:
        log.message('Deleting Fields')
        for i in fieldList:
            arcpy.DeleteField_management(eraseOutput, i)
            log.message(i + ' was deleted')
    except:
        log.message('No fields Deleted')


delfields()
