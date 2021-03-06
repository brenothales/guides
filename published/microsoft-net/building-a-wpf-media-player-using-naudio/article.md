Last year I needed to build a **W**indows **P**resentation **F**oundation (WPF) application for an electronic stethoscope which is able to record respiratory audio, save them to wave files, and if user wants, play the wave files at a later time. At this point my only experience with audio in general was with Unity3D -which has some great tools for it- and Matlab back in my university days. I remember thinking to myself "How hard can it be? I already know C# can play wave files, there must be some advanced tools in the core libraries!". I was, of course, very, *very* wrong.

Not only I was shocked to find no core libraries, I was also amazed how audio is a very deep and challenging subject in programming. Lucky for me, Pluralsight has great courses on [audio in general](https://www.pluralsight.com/courses/digital-audio-fundamentals) and on [NAudio](https://www.pluralsight.com/courses/audio-programming-naudio), so I was able to finish my project successfully.

I believe, to learn something properly, you need to create a basic project using it. So in this tutorial I will explain how to build a simple media player from ground up using a very popular .NET audio library NAudio.

### Why Build Your Own Media Player?
Other than the fact that it is fun:
- You might have a project where you need to make a in-app media player to achieve your goals like my situation.
- You might need a very special feature that no other media player provides.
- You might also not trust 3rd party media players in your secure environment where you absolutely have to use a media player.

Whatever your reason is, you have challenges waiting for you down the road when it comes to audio programming.

### Challenges
The first challenge is the concept that audio is not a traditional object, but a stream object. A stream is usually a sequence of bytes. If you want to read it, it has to be read like an array. Also it can't be manipulated easily, you have to dig into the byte array and figure out what to do with each byte to achieve what you want. They are not very easy to debug either. There are also considerations when it comes to audio such as, number of channels, frequency, file formats etc. Especially when it comes to debugging it can be a mess.

There are questions need answering such as "How do we know if we reach the end of an audio file?", "How do we stop or pause the stream?", "How do we resume where we left off?". Of course there are many more issues you need to address but these are the core issues.

Finally there are memory management issues. Typically, when you are done with a stream you have to dispose of it properly otherwise it will stay in the memory causing a memory leak. For example, if you are working with hundreds of audio and you don't dispose them properly, you could run out of memory really fast and your application may crash. So in essence, you have to manually manage memory.

So when you think about audio considering these challenges, even a simple action such as playing an audio file, gets very complicated very fast.

### NAudio to the Rescue
Lucky for us, there is an audio library for .NET - NAudio - that will do most of this work for us. We will still be working with streams and we will have to face several challenges that comes with them, however NAudio is a great library that abstracts us from the details of simple playing and recording of audio files. The good thing about NAudio libarary is, it can be used in every type of project. If you simply want build and application to play or record audio files, you can do that without delving deep into streams. If you want something complex such as manipulating audio or creating filters it provides very good tools for that too.

There are 3 ways of getting NAudio:
- [NAudio codeplex page](https://naudio.codeplex.com/)
- [NAudio GitHub page](https://github.com/naudio/NAudio)
- And finally via [NuGet](https://www.nuget.org/packages/NAudio/)

I usually go with NuGet since it is the easiest of them all, however if you want to see how it actually works there are great examples and documentation on the Codeplex and GitHub pages.

### Media Player

The main purpose of this tutorial is to build a simple media player which will have some common features of various media players such as:
- Play audio files with various formats (wav, mp3, ogg, flac etc)
- Skip to beginning, skip to end, play/pause, stop, shuffle buttons and their functionality
- Have a volume control
- Have a seekbar
- Have a playlist
- Add a file to this playlist
- Add a folder of files to this playlist
- Save and load playlists
- When a track finishes, it will move to the next track in the playlist automatically.

It should look like this at the end (with my Witcher 3 soundtrack playlist):
![Our finalized media player](https://raw.githubusercontent.com/pluralsight/guides/master/images/71e4ccf5-4301-4b3f-8cb2-7fde9b7a816e.png)

### Implementation
In the solution, we will have two projects. One for the actual application and one for the NAudio abstraction. NAudio is great in abstracting us from details but I want to make it very easy for us to do everything with a few methods, rather than calling NAudio codes for playing, pausing, stopping etc.

However there is a choice we have to make when it comes to the main project where the UI is; do we use the traditional event driven architecture or do we use **M**odel **V**iew **V**iew**M**odel architecture (MVVM). You might think that you would use MVVM because that's all the cool kids use nowadays, however both architectures have advantages and disadvantages especially when it comes to developing a real time application such as this. I developed media players using both architectures on two different projects:
- If you use MVVM you write way less code, however in this case, property binding has a nasty habit of breaking down and causing bugs if you bind the wrong way. Especially when it comes to implementing a real time changing two-way bound seekbar control.
- If you use the good old event driven architecture you get to write a lot more **U**ser **I**nterface (UI) event code, however you will have full control over what happens during those events so it is easier to implement a real time control such as a seekbar.

Since this is a brand new project I would go with MVVM, so in this tutorial I will use the MVVM architecture. However since this is not an MVVM tutorial I won't go into details on how MVVM works.

So now that we choose our architecture we need to generate the right namespaces in our project. Our solution structure will be like this:
- Solution
  - Project: NaudioPlayer
    - Models
    - ViewModels
    - Views
    - Services
    - Images
  - Project: NaudioWrapper

#### Mocking up the UI
I always like to start with the UI part when it comes to WPF projects because it provides me with a feature list I need to implement in a visual way. So I will start coding by first generating a UI mock-up so we can see what features we need to code.

Before we start on the UI however, I will provide you the link for the images I used for the buttons so you will have them ready. For icons I used Google's free material design icons. You can get them from Google's material design [page](https://material.google.com/resources/sticker-sheets-icons.html#). When you download the icons, do a quick search of "play" or "pause" in the download directory to find the relevant icons.

We also need `System.Windows.Interactivity` and `Microsoft.Expression.Interaction` namespaces for our event bindings. I said we won't use event driven architecture but sometimes there is no way avoiding events. So these namespaces provide events the MVVM way. Adding these namespaces can be tricky and might not always work as you expect them to work. These namespaces actually come with Blend, a UI development tool for XAML based projects. So if you have Blend installed (it comes with the Visual Studio installer), you can find those DLL files from:

- For .NET 4.5+ `C:\Program Files (x86)\Microsoft SDKs\Expression\Blend\.NETFramework\v4.5\Libraries`
- For .NET 4.0 `C:\Program Files (x86)\Microsoft SDKs\Expression\Blend\.NETFramework\v4.0\Libraries`

If you don't have Blend installed just run the Visual Studio installer and modify/add it to your installation.

However, in my experience adding those in Visual Studio doesn't always work. So what I usually do:
- Create the WPF project in Visual Studio
- Add `xmlns:i="http://schemas.microsoft.com/expression/2010/interactivity"` to the `Window`.
- Close Visual Studio and open the project in Blend.
- From the menu click Project->Add Reference and add the references from that menu.
- Close Blend and open the project back in Visual Studio

Now everything is ready for our UI code.

Here is our E**x**tensible **A**pplication **M**arkup **L**anguage (XAML) code for the UI:
```xml
<Window x:Class="NaudioPlayer.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:NaudioPlayer"
        xmlns:i="http://schemas.microsoft.com/expression/2010/interactivity"
        xmlns:viewModels="clr-namespace:NaudioPlayer.ViewModels"
        mc:Ignorable="d"
        Title="{Binding Title}" Height="350" Width="525">
    <Window.DataContext>
        <viewModels:MainWindowViewModel/>
    </Window.DataContext>
    <DockPanel LastChildFill="True">
        <Menu DockPanel.Dock="Top">
            <MenuItem Header="File">
                <MenuItem Header="Save Playlist" Command="{Binding SavePlaylistCommand}"/>
                <MenuItem Header="Load Playlist" Command="{Binding LoadPlaylistCommand}"/>
                <MenuItem Header="Exit" Command="{Binding ExitApplicationCommand}"/>
            </MenuItem>
            <MenuItem Header="Media">
                <MenuItem Header="Add File to Playlist..." Command="{Binding AddFileToPlaylistCommand}"/>
                <MenuItem Header="Add Folder to Playlist..." Command="{Binding AddFolderToPlaylistCommand}"/>
            </MenuItem>
        </Menu>
        <Grid DockPanel.Dock="Bottom" Height="30">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="30"/>
                <ColumnDefinition Width="30"/>
                <ColumnDefinition Width="30"/>
                <ColumnDefinition Width="30"/>
                <ColumnDefinition Width="*"/>
                <ColumnDefinition Width="30"/>
            </Grid.ColumnDefinitions>
            <Button Grid.Column="0" Margin="3" Command="{Binding RewindToStartCommand}">
                <Image Source="../Images/skip_previous.png" VerticalAlignment="Center" HorizontalAlignment="Center"/>
            </Button>
            <Button Grid.Column="1" Margin="3" Command="{Binding StartPlaybackCommand}">
                <Image Source="{Binding PlayPauseImageSource}" VerticalAlignment="Center" HorizontalAlignment="Center"/>
            </Button>
            <Button Grid.Column="2" Margin="3" Command="{Binding StopPlaybackCommand}">
                <Image Source="../Images/stop.png" VerticalAlignment="Center" HorizontalAlignment="Center"/>
            </Button>
            <Button Grid.Column="3" Margin="3" Command="{Binding ForwardToEndCommand}">
                <Image Source="../Images/skip_next.png" VerticalAlignment="Center" HorizontalAlignment="Center"/>
            </Button>
            <Button Grid.Column="5" Margin="3" Command="{Binding ShuffleCommand}">
                <Image Source="../Images/shuffle.png" VerticalAlignment="Center" HorizontalAlignment="Center"/>
            </Button>
        </Grid>
        <Grid DockPanel.Dock="Bottom" Margin="3">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="3*"/>
                <ColumnDefinition Width="20"/>
                <ColumnDefinition Width="20"/>
                <ColumnDefinition Width="*"/>
            </Grid.ColumnDefinitions>
            <Slider Grid.Column="0" Minimum="0" Maximum="{Binding CurrentTrackLenght, Mode=OneWay}" Value="{Binding CurrentTrackPosition, Mode=TwoWay}" x:Name="SeekbarControl" VerticalAlignment="Center">
                <i:Interaction.Triggers>
                    <i:EventTrigger EventName="PreviewMouseDown">
                        <i:InvokeCommandAction Command="{Binding TrackControlMouseDownCommand}"></i:InvokeCommandAction>
                    </i:EventTrigger>
                    <i:EventTrigger EventName="PreviewMouseUp">
                        <i:InvokeCommandAction Command="{Binding TrackControlMouseUpCommand}"></i:InvokeCommandAction>
                    </i:EventTrigger>
                </i:Interaction.Triggers>
            </Slider>
            <Image Grid.Column="2" Source="../Images/volume.png"></Image>
            <Slider Grid.Column="3" Minimum="0" Maximum="1" Value="{Binding CurrentVolume, Mode=TwoWay}" x:Name="VolumeControl" VerticalAlignment="Center">
                <i:Interaction.Triggers>
                    <i:EventTrigger EventName="ValueChanged">
                        <i:InvokeCommandAction Command="{Binding VolumeControlValueChangedCommand}"></i:InvokeCommandAction>
                    </i:EventTrigger>
                </i:Interaction.Triggers>
            </Slider>
        </Grid>
        <Grid DockPanel.Dock="Bottom">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="70"/>
                <ColumnDefinition Width="Auto"/>
            </Grid.ColumnDefinitions>
            <TextBlock Grid.Column="0" Text="Now playing: "></TextBlock>
            <TextBlock Grid.Column="1" Text="{Binding CurrentlyPlayingTrack.FriendlyName, Mode=OneWay}"/>
        </Grid>
        <ListView x:Name="Playlist" ItemsSource="{Binding Playlist}" SelectedItem="{Binding CurrentlySelectedTrack, Mode=TwoWay}">
            <ListView.ItemTemplate>
                <DataTemplate>
                    <Grid>
                        <Grid.ColumnDefinitions>
                            <ColumnDefinition></ColumnDefinition>
                        </Grid.ColumnDefinitions>
                        <TextBlock Grid.Column="0" Text="{Binding Path=FriendlyName, Mode=OneWay}"></TextBlock>
                    </Grid>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
    </DockPanel>
</Window>

```

So by looking at our XAML code, in our ``MainWindowViewModel`` viewmodel, we need to implement the following commands

- Menu Commands
  - **SavePlaylistCommand:** This command will save our `Playlist` into a text file, simply by writing the path of each `Track` in the `Playlist` as `<filename>.playlist`.
  - **LoadPlaylistCommand:** This command will read the `.playlist` file of our choosing and generate a `Playlist` for us.
  - **ExitApplicationCommand:** This command will dispose all our audio streams and close the application.
  - **AddFileToPlaylistCommand:** This command will add a single audio file to our `Playlist`.
  - **AddFolderToPlaylistCommand:** This command will add a folder of files to our `Playlist`.
- Player Commands
  - **RewindToStartCommand:** This command will set the `CurrentTrackPosition` to 0, effectively skipping to start.
  - **StartPlaybackCommand:** This command will start playback of the `CurrentlySelectedTrack`. When pressed during playback, it will pause the playback.
  - **StopPlaybackCommand:** This command will stop the playback.
  - **ForwardToEndCommand:** This command will skip to the last second of the currently playing audio file.
  - **ShuffleCommand:** This command will shuffle our `Playlist` randomly.
- MVVM Events
  - **TrackControlMouseDownCommand:** This command will run when we press mouse button on the seekbar slider control.
  - **TrackControlMouseUpCommand:** This command will run when we release mouse button from the seekbar slider control.
  - **VolumeControlValueChangedCommand:** This command will run when we change the value of the volume control slider.

and properties:
- **Title:** Title of the window.
- **CurrentTrackPosition:** While the audio is playing, this property is used to store the current position in seconds.
- **CurrentVolume:** This property sets the volume.
- **Playlist:** This property represents our playlist.
- **CurrentlySelectedTrack:** This property represents currently selected track in our playlist.
- **CurrentlyPlayingTrack:** This property represents currently playing track in our playlist.
- **PlayPauseImageSource:** This property sets either play or pause images depending on whether the audio is playing or paused to our play button.

### NaudioWrapper
Before we move on to the ViewModel we need the basic features abstracted away from the core NAudio code. To do this, first we must add NAudio to this project via NuGet. Then we must create an `AudioPlayer` class to hold all these features in its methods and our ViewModel can access these public methods to interact with it.

Looking at our feature list, first and foremost, we need to actually play an audio file. To do this in NAudio, first we must set the file path of the audio and read the file with a reader. There are many specific readers for specific filetypes. However you can simply use `AudioFileReader` to read all supported files. To play the file, we need to set an output. We want to play all kinds of audio files so we will use `DirectSoundOut`. You can use many other output types, however we will use `DirectSoundOut` in this example.

We must also create some events to let ViewModel know that we are playing, paused or stopped. These events will come in handy to manipulate our UI accordingly.

And lastly, we must create a flag to set or get the stop type. This is needed because we need to know if we are stopping to play the next file in our playlist or if we are stopping because user wants to stop playing the current file.

```cs
public class AudioPlayer
{
    public enum PlaybackStopTypes
    {
        PlaybackStoppedByUser, PlaybackStoppedReachingEndOfFile
    }

    public PlaybackStopTypes PlaybackStopType { get; set; }
    
    private AudioFileReader _audioFileReader;

    private DirectSoundOut _output;

    private string _filepath;
    
    public event Action PlaybackResumed;
    public event Action PlaybackStopped;
    public event Action PlaybackPaused;
}
```

Let's take a look at our methods. In the constructor we must initialize all our fields and their events.
```cs
public AudioPlayer(string filepath, float volume)
{
    PlaybackStopType = PlaybackStopTypes.PlaybackStoppedReachingEndOfFile;

    _audioFileReader = new AudioFileReader(filepath) { Volume = volume };

    _output = new DirectSoundOut(200);
    _output.PlaybackStopped += _output_PlaybackStopped;

    var wc = new WaveChannel32(_audioFileReader);
    wc.PadWithZeroes = false;

    _output.Init(wc);
}
```
As default we go for `PlaybackStopTypes.PlaybackStoppedReachingEndOfFile` for our `PlaybackStopType`, because usually that's why our playback would stop. Here the most important bit is `wc.PadWithZeroes = false` because typically `PlaybackStopped` event fires if the reader finds byte value 0 and if we don't pad with zeros then playback would go on forever and there would be no way for us to know if the clip is finished.

Now that we initialized everything, we can implement the rest of our methods. We need methods for:
- `PlaybackStopped` event
- Playing
- Pausing
- Stopping
- Toggling between Play and Pause
- Getting and setting current track position for our UI
- Getting and setting current volume for our UI
- And a `Dispose()` method to clear up the memory

```cs
public void Play(PlaybackState playbackState, double currentVolumeLevel)
{
    if (playbackState == PlaybackState.Stopped || playbackState == PlaybackState.Paused)
    {
        _output.Play();
    }

    _audioFileReader.Volume = (float) currentVolumeLevel;

    if (PlaybackResumed != null)
    {
        PlaybackResumed();
    }
}

private void _output_PlaybackStopped(object sender, StoppedEventArgs e)
{
    Dispose();
    if (PlaybackStopped != null)
    {
        PlaybackStopped();
    }
}

public void Stop()
{
    if (_output != null)
    {
        _output.Stop();
    }
}

public void Pause()
{
    if (_output != null)
    {
        _output.Pause();

        if (PlaybackPaused != null)
        {
            PlaybackPaused();
        }
    }
}

public void TogglePlayPause(double currentVolumeLevel)
{
    if (_output != null)
    {
        if (_output.PlaybackState == PlaybackState.Playing)
        {
            Pause();
        }
        else
        {
            Play(_output.PlaybackState, currentVolumeLevel);
        }
    }
    else
    {
        Play(PlaybackState.Stopped, currentVolumeLevel);
    }
}

public void Dispose()
{
    if (_output != null)
    {
        if (_output.PlaybackState == PlaybackState.Playing)
        {
            _output.Stop();
        }
        _output.Dispose();
        _output = null;
    }
    if (_audioFileReader != null)
    {
        _audioFileReader.Dispose();
        _audioFileReader = null;
    }
}

public double GetLenghtInSeconds()
{
    if (_audioFileReader != null)
    {
        return _audioFileReader.TotalTime.TotalSeconds;
    }
    else
    {
        return 0;
    }
}

public double GetPositionInSeconds()
{
    return _audioFileReader != null ? _audioFileReader.CurrentTime.TotalSeconds : 0;
}

public float GetVolume()
{
    if (_audioFileReader != null)
    {
        return _audioFileReader.Volume;
    }
    return 1;
}

public void SetPosition(double value)
{
    if (_audioFileReader != null)
    {
        _audioFileReader.CurrentTime = TimeSpan.FromSeconds(value);
    }
}

public void SetVolume(float value)
{
    if (_output != null)
    {
        _audioFileReader.Volume = value;
    }
}
```

### The ViewModel
After coding our abstraction layer, we can move on the ViewModel. However, before we implement the ViewModel we need to define how commands work as in every MVVM project. We will use the commonly used `RelayCommand` class to do this:
```cs
public class RelayCommand : ICommand
{
    private readonly Action<object> _execute;
    private readonly Predicate<object> _canExecute;

    public RelayCommand(Action<object> execute, Predicate<object> canExecute)
    {
        _execute = execute;
        _canExecute = canExecute;
    }
    public bool CanExecute(object parameter)
    {
        bool result = _canExecute == null ? true : _canExecute(parameter);
        return result;
    }

    public void Execute(object parameter)
    {
        _execute(parameter);
    }

    public event EventHandler CanExecuteChanged
    {
        add { CommandManager.RequerySuggested += value; }
        remove { CommandManager.RequerySuggested -= value; }
    }
}
```

Now, let's stub out our ViewModel:

```cs
public class MainWindowViewModel : INotifyPropertyChanged
{
    private string _title;
    private double _currentTrackLenght;
    private double _currentTrackPosition;
    private string _playPauseImageSource;
    private float _currentVolume;

    private ObservableCollection<Track> _playlist;
    private Track _currentlyPlayingTrack;
    private Track _currentlySelectedTrack;
    private AudioPlayer _audioPlayer;

    public string Title
    {
        get { return _title; }
        set
        {
            if (value == _title) return;
            _title = value;
            OnPropertyChanged(nameof(Title));
        }
    }

    public string PlayPauseImageSource
    {
        get { return _playPauseImageSource; }
        set
        {
            if (value == _playPauseImageSource) return;
            _playPauseImageSource = value;
            OnPropertyChanged(nameof(PlayPauseImageSource));
        }
    }

    public float CurrentVolume
    {
        get { return _currentVolume; }
        set
        {

            if (value.Equals(_currentVolume)) return;
            _currentVolume = value;
            OnPropertyChanged(nameof(CurrentVolume));
        }
    }

    public double CurrentTrackLenght
    {
        get { return _currentTrackLenght; }
        set
        {
            if (value.Equals(_currentTrackLenght)) return;
            _currentTrackLenght = value;
            OnPropertyChanged(nameof(CurrentTrackLenght));
        }
    }

    public double CurrentTrackPosition
    {
        get { return _currentTrackPosition; }
        set
        {
            if (value.Equals(_currentTrackPosition)) return;
            _currentTrackPosition = value;
            OnPropertyChanged(nameof(CurrentTrackPosition));
        }
    }

    public Track CurrentlySelectedTrack
    {
        get { return _currentlySelectedTrack; }
        set
        {
            if (Equals(value, _currentlySelectedTrack)) return;
            _currentlySelectedTrack = value;
            OnPropertyChanged(nameof(CurrentlySelectedTrack));
        }
    }

    public Track CurrentlyPlayingTrack
    {
        get { return _currentlyPlayingTrack; }
        set
        {
            if (Equals(value, _currentlyPlayingTrack)) return;
            _currentlyPlayingTrack = value;
            OnPropertyChanged(nameof(CurrentlyPlayingTrack));
        }
    }      

    public ObservableCollection<Track> Playlist
    {
        get { return _playlist; }
        set
        {
            if (Equals(value, _playlist)) return;
            _playlist = value;
            OnPropertyChanged(nameof(Playlist));
        }
    }

    public ICommand ExitApplicationCommand { get; set; }
    public ICommand AddFileToPlaylistCommand { get; set; }
    public ICommand AddFolderToPlaylistCommand { get; set; }
    public ICommand SavePlaylistCommand { get; set; }
    public ICommand LoadPlaylistCommand { get; set; }

    public ICommand RewindToStartCommand { get; set; }
    public ICommand StartPlaybackCommand { get; set; }
    public ICommand StopPlaybackCommand { get; set; }
    public ICommand ForwardToEndCommand { get; set; }
    public ICommand ShuffleCommand { get; set; }

    public ICommand TrackControlMouseDownCommand { get; set; }
    public ICommand TrackControlMouseUpCommand { get; set; }
    public ICommand VolumeControlValueChangedCommand { get; set; }

    public event PropertyChangedEventHandler PropertyChanged;

    public MainWindowViewModel()
    {		
        LoadCommands();
		
        Playlist = new ObservableCollection<Track>();            
		
		Title = "NaudioPlayer";
        PlayPauseImageSource = "../Images/play.png";            
    }       

    private void LoadCommands()
    {
        // Menu commands
        ExitApplicationCommand = new RelayCommand(ExitApplication,CanExitApplication);
        AddFileToPlaylistCommand = new RelayCommand(AddFileToPlaylist, CanAddFileToPlaylist);
        AddFolderToPlaylistCommand = new RelayCommand(AddFolderToPlaylist, CanAddFolderToPlaylist);
        SavePlaylistCommand = new RelayCommand(SavePlaylist, CanSavePlaylist);
        LoadPlaylistCommand = new RelayCommand(LoadPlaylist, CanLoadPlaylist);

        // Player commands
        RewindToStartCommand = new RelayCommand(RewindToStart, CanRewindToStart);
        StartPlaybackCommand = new RelayCommand(StartPlayback, CanStartPlayback);
        StopPlaybackCommand = new RelayCommand(StopPlayback, CanStopPlayback);
        ForwardToEndCommand = new RelayCommand(ForwardToEnd, CanForwardToEnd);
        ShuffleCommand = new RelayCommand(Shuffle, CanShuffle);

        // Event commands
        TrackControlMouseDownCommand = new RelayCommand(TrackControlMouseDown, CanTrackControlMouseDown);
        TrackControlMouseUpCommand = new RelayCommand(TrackControlMouseUp, CanTrackControlMouseUp);
        VolumeControlValueChangedCommand = new RelayCommand(VolumeControlValueChanged, CanVolumeControlValueChanged);
    }

    // Menu commands
    private void ExitApplication(object p)
    {
        
    }
    private bool CanExitApplication(object p)
    {
        return true;
    }

    private void AddFileToPlaylist(object p)
    {
        
    }
    private bool CanAddFileToPlaylist(object p)
    {
        return true;
    }

    private void AddFolderToPlaylist(object p)
    {
        
    }

    private bool CanAddFolderToPlaylist(object p)
    {
        return true;
    }

    private void SavePlaylist(object p)
    {
        
    }

    private bool CanSavePlaylist(object p)
    {
        return true;
    }

    private void LoadPlaylist(object p)
    {
        
    }

    private bool CanLoadPlaylist(object p)
    {
        return true;
    }

    // Player commands
    private void RewindToStart(object p)
    {
        
    }
    private bool CanRewindToStart(object p)
    {
        return true;
    }

    private void StartPlayback(object p)
    {
        
    }

    private bool CanStartPlayback(object p)
    {
        return true;
    }

    private void StopPlayback(object p)
    {
        
    }
    private bool CanStopPlayback(object p)
    {
        return true;
    }

    private void ForwardToEnd(object p)
    {
        
    }
    private bool CanForwardToEnd(object p)
    {
        return true;
    }

    private void Shuffle(object p)
    {
        
    }
    private bool CanShuffle(object p)
    {
        return true;
    }

    // Events
    private void TrackControlMouseDown(object p)
    {
        
    }

    private void TrackControlMouseUp(object p)
    {
        
    }

    private bool CanTrackControlMouseDown(object p)
    {
        return true;
    }

    private bool CanTrackControlMouseUp(object p)
    {
        return true;
    }

    private void VolumeControlValueChanged(object p)
    {
        
    }

    private bool CanVolumeControlValueChanged(object p)
    {
        return true;
    }

    [NotifyPropertyChangedInvocator]
    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```
##### Exiting the Application
Let's start with the easy but the important one, exiting our application. We must consider two ways to quit:
- User clicks exit in the menu
- User clicks the red X

For the menu part:
```cs
private void ExitApplication(object p)
{
    if (_audioPlayer != null)
    {
        _audioPlayer.Dispose();
    }
    
    Application.Current.Shutdown();
}
private bool CanExitApplication(object p)
{
    return true;
}
```

Here we must manage the memory in case user played some audio clips and `AudioPlayer` might not be null. Even if we close the application audio clip as a stream would remain in the memory so we have to `Dispose()` it to release the memory.

For the red X button, first we must add an event in the constructor, letting ViewModel know that we are exiting.

```cs
public MainWindowViewModel()
{
    Application.Current.MainWindow.Closing += MainWindow_Closing;

    Title = "NaudioPlayer";

    LoadCommands();

    Playlist = new ObservableCollection<Track>();

    PlayPauseImageSource = "../Images/play.png";
    CurrentVolume = 1;
}
```

Then add the method:
```cs
private void MainWindow_Closing(object sender, CancelEventArgs e)
{
    if (_audioPlayer != null)
    {
        _audioPlayer.Dispose();
    }
}
```
##### Adding Files to Playlist
Now, let's move on to creating our playlist by adding a single file. However we do not want to modify our playlist while a clip is running. So we need a flag to see if we are playing a clip. We must add another field and modify the constructor for this.
```cs
private enum PlaybackState
{
    Playing, Stopped, Paused
}

private PlaybackState _playbackState;

public MainWindowViewModel()
{
    Application.Current.MainWindow.Closing += MainWindow_Closing;

    Title = "NaudioPlayer";

    LoadCommands();

    Playlist = new ObservableCollection<Track>();
    
    _playbackState = PlaybackState.Stopped;

    PlayPauseImageSource = "../Images/play.png";
    CurrentVolume = 1;
}
```
And to add a file, we do it with an `OpenFileDialog`:
```cs
private void AddFileToPlaylist(object p)
{
    var ofd = new OpenFileDialog();
    ofd.Filter = "Audio files (*.wav, *.mp3, *.wma, *.ogg, *.flac) | *.wav; *.mp3; *.wma; *.ogg; *.flac";
    var result = ofd.ShowDialog();
    if (result == true)
    {
        var friendlyName = ofd.SafeFileName.Remove(ofd.SafeFileName.Length - 4);
        var track = new Track(ofd.FileName, friendlyName);
        Playlist.Add(track);
    }
}
private bool CanAddFileToPlaylist(object p)
{
    if (_playbackState == PlaybackState.Stopped)
    {
        return true;
    }
    return false;
}
```

We also need to implement add a folder of files. To do this first we need a way to select a folder like we did with the file with `CommonOpenFileDialog()`. However core libraries do not have this class so we need to add this class via NuGet. The name of he package is Microsoft.WindowsAPICodePack-Core. This code can only be used on machines with Vista or above.

```cs
private void AddFolderToPlaylist(object p)
{
    var cofd = new CommonOpenFileDialog();
    cofd.IsFolderPicker = true;
    var result = cofd.ShowDialog();
    if (result == CommonFileDialogResult.Ok)
    {
        var folderName = cofd.FileName;
        var audioFiles = Directory.EnumerateFiles(folderName, "*.*", SearchOption.AllDirectories)
                                  .Where(f=>f.EndsWith(".wav") || f.EndsWith(".mp3") || f.EndsWith(".wma") || f.EndsWith(".ogg") || f.EndsWith(".flac"));
        foreach (var audioFile in audioFiles)
        {
            var removePath = audioFile.RemovePath();
            var friendlyName = removePath.Remove(removePath.Length - 4);
            var track = new Track(audioFile, friendlyName);
            Playlist.Add(track);
        }
        Playlist = new ObservableCollection<Track>(Playlist.OrderBy(z => z.FriendlyName).ToList());
    }
}

private bool CanAddFolderToPlaylist(object p)
{
    if (_playbackState == PlaybackState.Stopped)
    {
        return true;
    }
    return false;
}
```

##### Saving and Loading The Playlist
To even add more to playlist functionality we must be able to save and load our playlists. Let's choose ".playlist" for our file extension.

To save a playlist:
```cs
private void SavePlaylist(object p)
{
    var sfd = new SaveFileDialog();
    sfd.CreatePrompt = false;
    sfd.OverwritePrompt = true;
    sfd.Filter = "PLAYLIST files (*.playlist) | *.playlist";
    if (sfd.ShowDialog() == true)
    {
        var ps = new PlaylistSaver();
        ps.Save(Playlist, sfd.FileName);
    }
}

private bool CanSavePlaylist(object p)
{
    return true;
}
```

To load a playlist:
```cs
private void LoadPlaylist(object p)
{
    var ofd = new OpenFileDialog();
    ofd.Filter = "PLAYLIST files (*.playlist) | *.playlist";
    if (ofd.ShowDialog() == true)
    {
        Playlist = new PlaylistLoader().Load(ofd.FileName).ToObservableCollection();
    }
}

private bool CanLoadPlaylist(object p)
{
    return true;
}
```

We are finally done with our menu commands, we can move to the player buttons that deals with the actual audio.

##### Skip to Beginning
To skip to beginning, we need to set the current track position to zero while the audio is playing. The code for that is:

```cs
private void RewindToStart(object p)
{
    _audioPlayer.SetPosition(0);
}
private bool CanRewindToStart(object p)
{
    if (_playbackState == PlaybackState.Playing)
    {
        return true;
    }
    return false;
}
```

##### Toggle for Play/Pause
In this part we need to decide if we are playing or pausing. If we are stopped, it is easy, we just need to instantiate another `AudioPlayer` and play the clip. However if we are playing or in pause mode, we need to check if the playing clip is the same as selected clip on the UI and then call `TogglePlayPause()` to resume or pause the clip.
```cs
private void StartPlayback(object p)
{
    if (CurrentlySelectedTrack != null)
    {
        if (_playbackState == PlaybackState.Stopped)
        {
            _audioPlayer = new AudioPlayer(CurrentlySelectedTrack.Filepath, CurrentVolume);
            _audioPlayer.PlaybackStopType = AudioPlayer.PlaybackStopTypes.PlaybackStoppedReachingEndOfFile;
            _audioPlayer.PlaybackPaused += _audioPlayer_PlaybackPaused;
            _audioPlayer.PlaybackResumed += _audioPlayer_PlaybackResumed;
            _audioPlayer.PlaybackStopped += _audioPlayer_PlaybackStopped;
            CurrentTrackLenght = _audioPlayer.GetLenghtInSeconds();
            CurrentlyPlayingTrack = CurrentlySelectedTrack;
        }
        if (CurrentlySelectedTrack == CurrentlyPlayingTrack)
        {
            _audioPlayer.TogglePlayPause(CurrentVolume);
        }
    }
}
private bool CanStartPlayback(object p)
{
    if (CurrentlySelectedTrack != null)
    {
        return true;
    }
    return false;
}
```
##### Events for the AudioPlayer
You might have noticed when we instantiated an `AudioPlayer`, we also subscribed to some events. These events are vital for the UI to know what "mode" we are in. In all 3 methods, We first set the `PlaybackState`, then load the correct image for the "play" button. However in `PlaybackStopped` event we will have to also refresh UI by `CommandManager.InvalidateRequerySuggested()` and set the current track position to zero indicating we finished the playback. Also if the playback is finished because we reached the end of a clip, we need to start playing the next clip. To find the next item, we need to setup an extension method for it.
```cs
public static T NextItem<T>(this ObservableCollection<T> collection, T currentItem)
{
    var currentIndex = collection.IndexOf(currentItem);
    if (currentIndex < collection.Count - 1)
    {
        return collection[currentIndex + 1];
    }
    return collection[0];
}
```

```cs
private void _audioPlayer_PlaybackStopped()
{
    _playbackState = PlaybackState.Stopped;
    PlayPauseImageSource = "../Images/play.png";
    CommandManager.InvalidateRequerySuggested();
    CurrentTrackPosition = 0;
    
    if (_audioPlayer.PlaybackStopType == AudioPlayer.PlaybackStopTypes.PlaybackStoppedReachingEndOfFile)
    {
        CurrentlySelectedTrack = Playlist.NextItem(CurrentlyPlayingTrack);
        StartPlayback(null);
    }
}

private void _audioPlayer_PlaybackResumed()
{
    _playbackState = PlaybackState.Playing;
    PlayPauseImageSource = "../Images/pause.png";
}

private void _audioPlayer_PlaybackPaused()
{
    _playbackState = PlaybackState.Paused;
    PlayPauseImageSource = "../Images/play.png";
}
```
##### Stop
For stopping first we need to indicate why we are stopping. We are stopping because user clicked stop or we are stopping because we reached the end of the current clip. Then stop the current clip and dispose it.
```cs
private void StopPlayback(object p)
{
    if (_audioPlayer != null)
    {
        _audioPlayer.PlaybackStopType = AudioPlayer.PlaybackStopTypes.PlaybackStoppedByUser;
        _audioPlayer.Stop();
    }
}
private bool CanStopPlayback(object p)
{
    if (_playbackState == PlaybackState.Playing || _playbackState == PlaybackState.Paused)
    {
        return true;
    }
    return false;
}
```

##### Skip to Next Track
When we reach to the end of the track we automatically move to the next track, however with this button we can skip to the end of the current track to trigger the move. We do this by setting the position to the last second of the current track.
```cs
private void ForwardToEnd(object p)
{
    if (_audioPlayer != null)
    {
        _audioPlayer.PlaybackStopType = AudioPlayer.PlaybackStopTypes.PlaybackStoppedReachingEndOfFile;
        _audioPlayer.SetPosition(_audioPlayer.GetLenghtInSeconds());
    }
}
private bool CanForwardToEnd(object p)
{
    if (_playbackState == PlaybackState.Playing)
    {
        return true;
    }
    return false;
}
```

##### Shuffle
First we need to make an extension method for `ObservableCollection<T>`. We will use a commonly used shuffle method from [this StackOverflow post](http://stackoverflow.com/questions/5383498/shuffle-rearrange-randomly-a-liststring).

```cs
public static ObservableCollection<T> Shuffle<T>(this ObservableCollection<T> input)
{
    var provider = new RNGCryptoServiceProvider();
    var n = input.Count;
    while (n > 1)
    {
        var box = new byte[1];
        do provider.GetBytes(box);
        while (!(box[0] < n * (Byte.MaxValue / n)));
        var k = (box[0] % n);
        n--;
        var value = input[k];
        input[k] = input[n];
        input[n] = value;
    }

    return input;
}
```
And for the command:
```cs
private void Shuffle(object p)
{
    Playlist = Playlist.Shuffle();
}
private bool CanShuffle(object p)
{
    if (_playbackState == PlaybackState.Stopped)
    {
        return true;
    }
    return false;
}
```

For the last part we need to deal with our MVVM event commands.

##### PreviewMouseDown and PreviewMouseUp Events on Seekbar
Here, we need to think what we are actually doing while we are scrubbing left and right in the seekbar. First we click and hold our mouse button, then while button is held we move and release the button. So here we need two events; one is for `MouseDown` and the other is `MouseUp`. While we are holding the mouse key we need to pause the clip, when we release we set the current seekbar value to the current track's position to move the clip there.

```cs
private void TrackControlMouseDown(object p)
{
    if (_audioPlayer != null)
    {
        _audioPlayer.Pause();
    }
}

private void TrackControlMouseUp(object p)
{
    if (_audioPlayer != null)
    {
        _audioPlayer.SetPosition(CurrentTrackPosition);
        _audioPlayer.Play(NAudio.Wave.PlaybackState.Paused, CurrentVolume);
    }
}

private bool CanTrackControlMouseDown(object p)
{
    if (_playbackState == PlaybackState.Playing)
    {
        return true;
    }
    return false;
}

private bool CanTrackControlMouseUp(object p)
{
    if (_playbackState == PlaybackState.Paused)
    {
        return true;
    }
    return false;
}
```

##### Volume Control Event
Volume control code is very similar to the seekbar but it is way easier because we can just change the volume without needing to know where we are on the track. We just set the current value of the slider to the volume itself.
```cs
private void VolumeControlValueChanged(object p)
{
    if (_audioPlayer != null)
    {
        _audioPlayer.SetVolume(CurrentVolume);
    }
}

private bool CanVolumeControlValueChanged(object p)
{
    return true;
}
```

Finally, we finished developing a fully functional media player!

### Conclusion

In this tutorial, we built a media player using NAudio library from scratch. You can test it by loading your favorite music folder and play it. I hope this guide will be useful for your audio projects. Please feel free to post ideas and feedback.

Have fun and happy coding!



