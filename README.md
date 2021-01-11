# SleepBoxAnalysisReferences


# START:

%% Sleep Box Analysis;
% Code that Generates Reference File;
cd('C:\Users\ThalamusLab\Desktop\MATLAB Behavioral Tests\Codes\');


%% Define Cell Array;
Reference_Table = cell(1,1);
SleepBoutsWithStimulationINDEXING_Table = cell(1,1);
SleepBoutsWithOutStimulationINDEXING_Table = cell(1,1);


%% Pathway to Files;
addpath('C:\Users\ThalamusLab\Desktop\MATLAB Behavioral Tests\DATA');
addpath('C:\Users\ThalamusLab\Desktop\MATLAB Behavioral Tests\VIDEOS\');


%% Files;
prompt = {'Rat Number','Session Date','Sleep Box','Room number'};
dboxTitle = 'Input for analysis';
definput = {'RAT37','-6-6-20','-SB1','-R1'};
dims = [1,100];
opts.Interpreter = 'tex';
answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
RatNumber = answer1{1,1};
SessionDate = answer1{2,1};
SleepBox = answer1{3,1};
RoomNumber = answer1{4,1};
%% Data Files;
VideoFiles = strcat(RatNumber,SessionDate,SleepBox,RoomNumber,'-Vid.avi');
ConfirmSignalsVideoFiles = strcat(RatNumber,SessionDate,SleepBox,RoomNumber,'-Confirm-Vid.avi');
TimestampFiles = strcat(RatNumber,SessionDate,SleepBox,RoomNumber,'-Confirm-TIMESTAMP.csv');
ConfirmSignalsFiles = strcat(RatNumber,SessionDate,SleepBox,RoomNumber,'-Confirm-TIMESTAMP.csv');
VelocityFiles = strcat(RatNumber,SessionDate,SleepBox,RoomNumber,'-Velocity.csv');


%% Initialize References;
RatEntersSleepBox = 0;
FirstBout = 0;
EndOfStimulation = 0;
EndOfBoutAtEndOfStimulation = 0;
FirstBoutAfterStimulation = 0;
LastConfirmSignal = 0;
RatLeavesSleepBox = 0;

%% Pull File Timestamp File;
Pull_TimestampFile = TimestampFiles;
Pull_TimestampFile = readtable(Pull_TimestampFile,'ReadVariableNames',false);


%% Load Data Seconds;
LoadData_TimestampFile_Seconds = char(Pull_TimestampFile.Var3);
k = strfind(LoadData_TimestampFile_Seconds(1,:),'-');  % remove - and the two digits after
LoadData_TimestampFile_Seconds(:,k:k + 2) = [];
raw_timestamp_seconds1= str2num(LoadData_TimestampFile_Seconds);  % convert back to numbers

%% Convert Seconds to Milliseconds;
Seconds_to_Milliseconds = raw_timestamp_seconds1 * 1000;

%% Load Data Minutes;
LoadData_TimestampFile_Minutes = Pull_TimestampFile.Var2;
% Carmen's Code for Minutes:
if length(LoadData_TimestampFile_Minutes) < 108000
%% Accumulate Minutes (Minutes Resets after 59 to 0; add 60 to Correct this);
Minutes = zeros(length(LoadData_TimestampFile_Minutes),1);
% find indexes where each hour ends (resets):
HourIdx = find(diff(LoadData_TimestampFile_Minutes)<0); 
% because first hour is unlikely to start exactly at zero, work with it
% separetely
Minutes(1:HourIdx(1),1)= LoadData_TimestampFile_Minutes(1:HourIdx(1)); % first minute doesn't need adjustment
add = 60;
for i = 1:size(HourIdx,1) - 1   
    MinuteSection = LoadData_TimestampFile_Minutes(HourIdx(i) + 1:HourIdx(i + 1),1);
    Minutes(HourIdx(i) + 1:HourIdx(i + 1),1) = MinuteSection + add; % add 60 seconds
    add = add + 60;
end    
% because the last hour is also unlikely to end exactly at minute 60, work with it
% separetely:
Minutes(HourIdx(i + 1) + 1:end,1) =  LoadData_TimestampFile_Minutes(HourIdx(i + 1) + 1:end,1) + add; 
% My Code for Minutes:
else
Minutes = zeros(length(LoadData_TimestampFile_Minutes),1);
offsetMinutes = 0;
for f = 2:length(LoadData_TimestampFile_Minutes(:,1))
   thisValue = LoadData_TimestampFile_Minutes((f),1);
   thisPreviousValue = LoadData_TimestampFile_Minutes((f - 1),1);
   if thisValue < thisPreviousValue
      offsetMinutes = offsetMinutes + 60; 
   end
   thisValue = thisValue + offsetMinutes;
   if f == 2
      Minutes((f - 1),1) = LoadData_TimestampFile_Minutes((f - 1),1);
   end
   Minutes((f),1) = thisValue;
end
end


%% Convert Minutes to Milliseconds;
Minutes_to_Milliseconds = Minutes*60000;

%% TimeStamp;
Timestamp = Minutes_to_Milliseconds + Seconds_to_Milliseconds;

%% TimeStamp Zeroed;
Timestamp_Zeroed = Timestamp - Timestamp(1,1);

%% Pull Velocity File;
% Pull_VelocityFile = VelocityFiles(1,(filesFromThisTestDay));
% Pull_VelocityFile = readtable(Pull_VelocityFile{1},'ReadVariableNames',false);
% Velocity = Pull_VelocityFile{:,1};
Pull_VelocityFile = VelocityFiles;
Velocity = readtable(Pull_VelocityFile,'ReadVariableNames',false);
Velocity = Velocity.Var1;

%% Find First Index When Rat Enters Sleep Box;
Video =  VideoFiles;
Video = VideoReader(Video);
h = 0; j = 0; i = 0;
while h == 0
   j = j + 1;
   thisValue = Velocity((j),1);
   if ~isnan(thisValue) == 1
      fig1 = figure;
      frames1 = read(Video,(j));
      frames1 = imshow(frames1);
      hold on;
      prompt = {'Enter:'};
      dboxTitle = 'the rat is present:';
      definput = {'no'};
      dims = [1,10];
      opts.Interpreter = 'tex';
      answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
      if strcmp(answer1,'no') == 0
         RatEntersSleepBox = j;
         h = h + 1;
         delete(fig1);
         close all hidden;
         close all force;
      else
         delete(fig1);
         close all hidden;
         close all force;
         k = 0;
         while k == 0
            j = j + 30;
            i = i + 1;
            fig1 = figure;
            frames1 = read(Video,(j));
            frames1 = imshow(frames1);
            hold on;
            prompt = {'Enter:'};
            dboxTitle = 'the rat is present:';
            definput = {'no'};
            dims = [1,10];
            opts.Interpreter = 'tex';
            answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
            if strcmp(answer1,'no') == 0
               RatEntersSleepBox = j;
               h = h + 1;
               k = k + 1;
               delete(fig1);
               close all hidden;
               close all force;
            elseif strcmp(answer1,'no') == 1 && i < 10
               delete(fig1);
               close all hidden;
               close all force;
            elseif strcmp(answer1,'no') == 1 && i == 10
               h = h + 1;
               k = k + 1;
               delete(fig1);
               close all hidden;
               close all force;
            end
         end
      end
   end
end

%% Stop here if there is an issue with the code;
if RatEntersSleepBox == 0
   fig1 = figure;
   hold on;
   prompt = {'Enter:'};
   dboxTitle = 'Issue With Coding';
   definput = {'ok'};
   dims = [1,10];
   opts.Interpreter = 'tex';
   answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
   delete(fig1);
   close all hidden;
   close all force;
else
%% Continue if there is no issue with the code;
   startIndexingHere = 0;
%% Pull Confirm Signals File;
% Pull_ConfirmSignalsFile = ConfirmSignalsFiles(1,(filesFromThisTestDay));
% Pull_ConfirmSignalsFile = readtable(Pull_ConfirmSignalsFile{1},'ReadVariableNames',false);
Pull_ConfirmSignalsFile = ConfirmSignalsFiles;
Pull_ConfirmSignalsFile = readtable(Pull_ConfirmSignalsFile,'ReadVariableNames',false);

%% Load Data Confirm Signals;
LoadData_ConfirmSignals = Pull_ConfirmSignalsFile.Var1;

%% Read String TRUE & FALSE Signals and Convert to 1's & 0's Because it's easier to work with;
% Carmen's Code for Converting the Confirm Signals file data;
FindFalse = strfind(LoadData_ConfirmSignals,'False');
IndexFalse = find(~cellfun(@isempty,FindFalse));
FindTrue = strfind(LoadData_ConfirmSignals,'True');
IndexTrue = find(~cellfun(@isempty,FindTrue));
ConfirmSignals = zeros(size(LoadData_ConfirmSignals,1),1);
ConfirmSignals(IndexTrue) = 1;

%% Start Indexing Here;
[row,column] = find(IndexTrue <= RatEntersSleepBox);
startIndexingHere = row(length(row(:,1)),1);

%% Determine First Bout of Sleep;
% Video =  VideoFiles(1,(filesFromThisTestDay));
% Video = VideoReader(Video{1});
Video =  VideoFiles;
Video = VideoReader(Video);
h = 0; z = 0; w = startIndexingHere - 1;
while h == 0
   w = w + 1;
   thisFrame = IndexTrue((w),1);
   fig1 = figure;
   frames1 = read(Video,(thisFrame));
   frames1 = imshow(frames1);
   prompt = {'Enter:'};
   dboxTitle = 'the Rat is present:';
   definput = {'no'};
   dims = [1,10];
   opts.Interpreter = 'tex';
   answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
   if strcmp(answer1,'no') == 0
      h = h + 1;
      delete(fig1);
      close all hidden;
      close all force;
%% Determine First Bout of Sleep;
      Video =  ConfirmSignalsVideoFiles;
      Video = VideoReader(Video);
      fig1 = figure;
      frames1 = read(Video,(thisFrame));
      frames1 = imshow(frames1);
      hold on;
      prompt = {'Enter:'};
      dboxTitle = 'the LED is:';
      definput = {'off'};
      dims = [1,10];
      opts.Interpreter = 'tex';
      answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
      if strcmp(answer1,'off') == 0
         FirstBout = w;
         h = h + 1;
         z = z + 1;
         delete(fig1);
         close all hidden;
         close all force;
      else
         h = h + 1;
         delete(fig1);
         close all hidden;
         close all force;
      end
      while z == 0
      w = w + 1;
      thisFrame = IndexTrue((w),1);
      fig1 = figure;
      frames1 = read(Video,(thisFrame));
      frames1 = imshow(frames1);
      hold on;
      prompt = {'Enter:'};
      dboxTitle = 'the LED is:';
      definput = {'off'};
      dims = [1,10];
      opts.Interpreter = 'tex';
      answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
      if strcmp(answer1,'off') == 0
         FirstBout = w;
         z = z + 1;
         delete(fig1);
         close all hidden;
         close all force;
      else
         delete(fig1);
         close all hidden;
         close all force;
      end
      end
   else
      delete(fig1);
      close all hidden;
      close all force;
   end
end


%% Determine End of Stimulation;
h = 0; i = 0; j = FirstBout;
while h == 0
   if j == length(IndexTrue(:,1))
      h = h + 1; 
   end
   j = j + 1;
   currentIndexedSignal = IndexTrue((j),1);
   currentIndexedSignal_plus_1 = IndexTrue((j + 1),1);
   currentIndexedSignal_plus_2 = IndexTrue((j + 2),1);
   if ((currentIndexedSignal_plus_1 - currentIndexedSignal) + (currentIndexedSignal_plus_2 - currentIndexedSignal_plus_1)) < 5 && i == 0
      EndOfStimulation = j;
      i = i + 1;
%% Determine End of Bout at End of Stimulation;
   elseif ((currentIndexedSignal_plus_1 - currentIndexedSignal) + (currentIndexedSignal_plus_2 - currentIndexedSignal_plus_1)) < 5 && i == 1
      i = i;
   elseif ((currentIndexedSignal_plus_1 - currentIndexedSignal) + (currentIndexedSignal_plus_2 - currentIndexedSignal_plus_1)) > 4 && i == 1
      EndOfBoutAtEndOfStimulation = j + 1;
      h = h + 1;
   end
   if j == length(IndexTrue(:,1))
      h = h + 1; 
   end
end


%% Determine First Bout After Stimulation;
% Video =  ConfirmSignalsVideoFiles(1,(filesFromThisTestDay));
% Video = VideoReader(Video{1});
Video =  ConfirmSignalsVideoFiles;
Video = VideoReader(Video);
h = 0; j = EndOfBoutAtEndOfStimulation;
while h == 0
   j = j + 1;
   thisIndex = IndexTrue((j),1);
   fig1 = figure;
   frames1 = read(Video,(thisIndex));
   frames1 = imshow(frames1);
   hold on;
   prompt = {'Enter:'};
   dboxTitle = 'the LED is:';
   definput = {'off'};
   dims = [1,10];
   opts.Interpreter = 'tex';
   answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
   if strcmp(answer1,'off') == 0
      FirstBoutAfterStimulation = j;
      h = h + 1;
      delete(fig1);
      close all hidden;
      close all force;
   else
      delete(fig1);
      close all hidden;
      close all force;
   end
end


%% Determine ON / OFF Signals During Bouts of Sleep When Stimulation;
% Video =  ConfirmSignalsVideoFiles(1,(filesFromThisTestDay));
% Video = VideoReader(Video{1});
Video =  ConfirmSignalsVideoFiles;
Video = VideoReader(Video);
IndexSleepBoutsWithStimulationSTART = [];
IndexSleepBoutsWithStimulationEND = [];
i = 0; e = 0;
Errors1 = 0;
for j = FirstBout:length(IndexTrue)
   thisIndex = IndexTrue((j),1);
   if i == 0
      if e == 0
         fig1 = figure;
         frames1 = read(Video,(thisIndex));
         frames1 = imshow(frames1);
         hold on;
         prompt = {'Enter:'};
         dboxTitle = 'the LED is:';
         definput = {'on'};
         dims = [1,10];
         opts.Interpreter = 'tex';
         answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
         if strcmp(answer1,'on') == 1
            IndexSleepBoutsWithStimulationSTART = [IndexSleepBoutsWithStimulationSTART;thisIndex];
            e = e + 1;
            delete(fig1);
            close all hidden;
            close all force;
         else
            Errors1 = Errors1 + 1;
            delete(fig1);
            close all hidden;
            close all force;
         end
      elseif e == 1
         fig1 = figure;
         frames1 = read(Video,(thisIndex));
         frames1 = imshow(frames1);
         hold on;
         prompt = {'Enter:'};
         dboxTitle = 'the LED is:';
         definput = {'off'};
         dims = [1,10];
         opts.Interpreter = 'tex';
         answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
         if strcmp(answer1,'off') == 1
            IndexSleepBoutsWithStimulationEND = [IndexSleepBoutsWithStimulationEND;thisIndex];
            e = e + 1;
            delete(fig1);
            close all hidden;
            close all force;
         else
            Errors1 = Errors1 + 1;
            delete(fig1);
            close all hidden;
            close all force;
         end
      end
   end
   if e == 2
      e = 0;
   end
   if j == EndOfStimulation
      i = i + 1;
   end
end


%% Determine the Last Confirm Signal When Rat is Still Present in the Sleep Box;
Video =  VideoFiles;
Video = VideoReader(Video);
h = 0; j = length(IndexTrue(:,1)) + 1;
while h == 0
   j = j - 1;
   thisIndex = IndexTrue((j),1);
   fig1 = figure;
   frames1 = read(Video,(thisIndex));
   frames1 = imshow(frames1);
   hold on;
   prompt = {'Enter:'};
   dboxTitle = 'the Rat is present:';
   definput = {'no'};
   dims = [1,10];
   opts.Interpreter = 'tex';
   answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
   if strcmp(answer1,'no') == 0
      LastConfirmSignal = j;
      h = h + 1;
      delete(fig1);
      close all hidden;
      close all force;
   else
      delete(fig1);
      close all hidden;
      close all force;
   end
end


%% Determine ON / OFF Signals During Bouts of Sleep After Stimulation is Applied;
IndexSleepBoutsWithOutStimulationSTART = [];
IndexSleepBoutsWithOutStimulationEND = [];
% Video =  ConfirmSignalsVideoFiles(1,(filesFromThisTestDay));
% Video = VideoReader(Video{1});
Video =  ConfirmSignalsVideoFiles;
Video = VideoReader(Video);
i = 0; e = 0; b = 0; m = 0;
Errors2 = 0;
for j = FirstBoutAfterStimulation:1:length(IndexTrue(:,1))
   thisIndex = IndexTrue((j),1);
   if m == 0
   if i == 0
      if e == 0
         fig1 = figure;
         frames1 = read(Video,(thisIndex));
         frames1 = imshow(frames1);
         hold on;
         prompt = {'Enter:'};
         dboxTitle = 'the LED is:';
         definput = {'on'};
         dims = [1,10];
         opts.Interpreter = 'tex';
         answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
         if strcmp(answer1,'on') == 1
            IndexSleepBoutsWithOutStimulationSTART = [IndexSleepBoutsWithOutStimulationSTART;thisIndex];
            e = e + 1;
            delete(fig1);
            close all hidden;
            close all force;
         else
            Errors2 = Errors2 + 1;
            delete(fig1);
            close all hidden;
            close all force;
         end
      elseif e == 1
         fig1 = figure;
         frames1 = read(Video,(thisIndex));
         frames1 = imshow(frames1);
         hold on;
         prompt = {'Enter:'};
         dboxTitle = 'the LED is:';
         definput = {'off'};
         dims = [1,10];
         opts.Interpreter = 'tex';
         answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
         if strcmp(answer1,'off') == 1
            IndexSleepBoutsWithOutStimulationEND = [IndexSleepBoutsWithOutStimulationEND;thisIndex];
            e = e + 1;
            delete(fig1);
            close all hidden;
            close all force;
         else
            Errors2 = Errors2 + 1;
            delete(fig1);
            close all hidden;
            close all force;
         end
      end
   end
   if e == 2
      e = 0;
%% Determine When Rat Leaves Sleep Box;
   if thisIndex == IndexTrue(LastConfirmSignal,1)
      if LastConfirmSignal == length(IndexTrue(:,1))
          RatLeavesSleepBox = IndexTrue(LastConfirmSignal,1);
          m = m + 1;
      else
      Video =  VideoFiles;
      Video = VideoReader(Video);
%% or Provide a General Estimate of When Rat Leaves Sleep Box;
      for u = thisIndex:150:IndexTrue(length(IndexTrue(:,1)),1)
         if b == 0
         fig1 = figure;
         frames1 = read(Video,(u));
         frames1 = imshow(frames1);
         hold on;
         prompt = {'Enter:'};
         dboxTitle = 'the rat is present:';
         definput = {'yes'};
         dims = [1,10];
         opts.Interpreter = 'tex';
         answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
         if strcmp(answer1,'yes') == 0
            RatLeavesSleepBox = u - 150;
            b = b + 1;
            delete(fig1);
            close all hidden;
            close all force;
         else
            if u >= IndexTrue(length(IndexTrue(:,1)),1)
               RatLeavesSleepBox = u;
               b = b + 1;
               delete(fig1);
               close all hidden;
               close all force;
            else
               delete(fig1);
               close all hidden;
               close all force;
            end
         end
         end
      end
      i = i + 1;
   end
   end
   end
end
end



%% Convert Indexes [IndexTrue(Index,1)] Back to Original Indexes [Exceptions: RatEntersSleepBox & RatLeavesSleepBox];
FirstBout = IndexTrue(FirstBout,1);
EndOfStimulation = IndexTrue(EndOfStimulation,1);
EndOfBoutAtEndOfStimulation = IndexTrue(EndOfBoutAtEndOfStimulation,1);
FirstBoutAfterStimulation = IndexTrue(FirstBoutAfterStimulation,1);
LastConfirmSignal = IndexTrue(LastConfirmSignal,1);
IndexSleepBoutsWithStimulationSTART = IndexTrue(IndexSleepBoutsWithStimulationSTART,1);
IndexSleepBoutsWithStimulationEND = IndexTrue(IndexSleepBoutsWithStimulationEND,1);
%% Update References Table;
Reference_Table{1,1} = [RatEntersSleepBox,FirstBout,EndOfStimulation,EndOfBoutAtEndOfStimulation,FirstBoutAfterStimulation,LastConfirmSignal,RatLeavesSleepBox];
%% Update Sleep Bouts Indexing Table;
SleepBoutsWithStimulationINDEXING_Table{1,1} = [IndexSleepBoutsWithStimulationSTART,IndexSleepBoutsWithStimulationEND];
SleepBoutsWithOutStimulationINDEXING_Table{1,1} = [IndexSleepBoutsWithOutStimulationSTART,IndexSleepBoutsWithOutStimulationEND];
%% Save References;
fig1 = figure;
hold on;
prompt = {'Great Work!'};
dboxTitle = 'Dialogue Box Will Self Destruct After Saving';
definput = {'save'};
dims = [1,100];
opts.Interpreter = 'tex';
answer1 = inputdlg(prompt,dboxTitle,dims,definput,opts);
if strcmp(answer1,'save') == 1
delete(fig1);
close all hidden;
close all force;
cd('C:\Users\ThalamusLab\Desktop\MATLAB Behavioral Tests\SAVE_DATA\');
%save('RAT37_REFERENCES.mat','Reference_Table');
save(strcat(RatNumber,SessionDate,'_REFERENCES.mat'),'Reference_Table');
%% Save Indexes of Sleep Bouts With Stimulation [START INDEXES, STOP INDEXES];
%save('RAT37_INDEXING_BOUTS_DURING_STIM.mat','SleepBoutsWithStimulationINDEXING_Table');
save(strcat(RatNumber,SessionDate,'_BOUTS_DURING_STIM.mat.mat'),'SleepBoutsWithStimulationINDEXING_Table');
%% Save Indexes of Sleep Bouts With Stimulation [START INDEXES, STOP INDEXES];
%save('RAT37_INDEXING_BOUTS_AFTER_STIM.mat','SleepBoutsWithOutStimulationINDEXING_Table');
save(strcat(RatNumber,SessionDate,'_BOUTS_AFTER_STIM.mat.mat'),'SleepBoutsWithOutStimulationINDEXING_Table');
else
    delete(fig1);
    close all hidden;
    close all force;
end
end



