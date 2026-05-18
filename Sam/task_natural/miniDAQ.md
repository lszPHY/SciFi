prescan=claude opus4.7
force model=deepseekv4pro
review model=gpt5.5
rank：2
tasakgroup： miniDAQ
commonstorage： rw
skills：local_env
##
I want to improve my current project,The target project is located at ~/phase2_MiniDAQ-no_scin_event_driven-GUI_DAQ
This is an ongoing project. Preserve existing work and continue from the current state.
find if there is any bug to be fixed, check
the fitted delta t and the delta calculated from PRC time。
The delta t should have a peak near 200 or 300ns， use decode in https://github.com/lszPHY/phase2_MiniDAQ.git as reference. Use python to plot the drift time figure.
and finaly tell me how many triger match event and check if the fitted and real trigger time is close enough
Use a local environment inside the project or task output directory.
Do not create or modify environments under /mnt/sci_envs/.
Prefer Python-only dependencies: numpy, scipy, matplotlib, pandas.
Do not install ROOT unless the existing project README explicitly requires it.
