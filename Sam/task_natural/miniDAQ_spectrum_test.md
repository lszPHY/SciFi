prescan=claude opus4.7
force model=deepseekv4pro
review model=gpt5.5
rank：2
taskgroup： miniDAQ
commonstorage： rw
skills：local_env
Timeout： 600

##
I want you to first read my project in https://github.com/lszPHY/phase2_MiniDAQ.git and sumrize anything that need to be noticed(like tube geometry, decode process, and how to plot ADC/TDC spectrum, etc) which continuing work agent need. Store this in taskgroup memory miniDAQ.
After you know key points, please read raw data located at /mnt/run00131_20260309_164054.dat , and plot the overall TDC spectrum TDC per channel ADC overall and ADC perchannel, and also the channel hits as did in my current project. Whenever you feel confused and don't know what to do, refer to my current project. You can clone to local.
I want to finally get figues that show the spectrum