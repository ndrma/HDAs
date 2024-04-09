#set the LOP / STAGE context
lop = hou.node('/stage')

#Start building the nodes, each has a variable so that they can be connected etc later
#they're not in a dictionary for transparency and easy swap of node archetype
#you can swap whichever you want, just follow the chain of nodes down the line for any new/removed

#sop import background
bg_import = lop.createNode('sopimport', 'background')

#background transform
transform_bg1 = lop.createNode('xform', 'background_transform')

#sop import for null
sop_import1 = lop.createNode('sopimport', 'FIRST_NULL')

#merge geos / sop imports
merge_geo1 = lop.createNode('merge')

#material library
matlib1 = lop.createNode('materiallibrary')

#material linker
matlinker1 = lop.createNode('materiallinker')

#camera
camera1 = lop.createNode('camera')

        ### LIGHT BRANCH START

#dome light / hdri
domelight1 = lop.createNode('domelight::2.0', 'hdri')

#area light 1 
arealight1 = lop.createNode('light::2.0')

#area light 2
arealight2 = lop.createNode('light::2.0')

#merge lights internal
merge_lights2 = lop.createNode('merge')

#light mixer
lightmixer1 = lop.createNode('lightmixer')

        ### LIGHT BRANCH END

#merge lights to main
merge_lights1 = lop.createNode('merge')

#karma render settings
karmarendersettings1 = lop.createNode('karmarenderproperties')

#usd render rop
usdrenderrop1 = lop.createNode('usdrender_rop')


#Wire them together
transform_bg1.setInput(0, bg_import, 0)
merge_geo1.setInput(0, transform_bg1, 0)
merge_geo1.setInput(1, sop_import1, 0)
matlib1.setInput(0, merge_geo1, 0)
matlinker1.setInput(0, matlib1, 0)
camera1.setInput(0, matlinker1, 0)
        #LIGHT BRANCH LINK START
merge_lights2.setInput(0, domelight1, 0)
merge_lights2.setInput(1, arealight1, 0)
merge_lights2.setInput(2, arealight2, 0)
lightmixer1.setInput(0,merge_lights2,0)
        #LIGHT BRANCH LINK END
merge_lights1.setInput(0, camera1, 0)
merge_lights1.setInput(1, lightmixer1, 0)
karmarendersettings1.setInput(0, merge_lights1, 0)
usdrenderrop1.setInput(0, karmarendersettings1, 0)

#set parameters for render nodes
karmarendersettings1.parm('engine').set("XPU Engine")
karmarendersettings1.parm('resolutionx').set(1920)
karmarendersettings1.parm('denoiser').set("nVidia Optix Denoiser")
usdrenderrop1.parm('renderer').set("Karma XPU")

#Align them
lop.layoutChildren()

#display/render flags
karmarendersettings1.setDisplayFlag(True)
