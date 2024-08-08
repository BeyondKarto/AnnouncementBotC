### Telegram YouTube Live Stream Notifier Bot

This Telegram bot monitors a specified YouTube channel for live streams and notifies users when a new live stream starts. The bot sends a notification with the live stream link and the first paragraph of the video description. After 5 minutes, the bot sends a follow-up message.

#### Features
- Monitors YouTube live streams.
- Sends notifications to a specified Telegram chat.
- Provides the live stream link and description.
- 
#### Prerequisites
- .NET 6.0 SDK or later
- Telegram Bot API Token
- YouTube Data API v3 Key

#### Installation
1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/TelegramYouTubeNotifierBot.git
   cd TelegramYouTubeNotifierBot
   ```

2. Install the necessary packages:
   ```bash
   dotnet add package Serilog
   dotnet add package Serilog.Sinks.Console
   dotnet add package Serilog.Sinks.File
   dotnet add package Telegram.Bot
   dotnet add package Google.Apis.YouTube.v3
   ```

3. Configure the bot:
   Update the `TELEGRAM_BOT_TOKEN`, `YOUTUBE_API_KEY`, and `YOUTUBE_CHANNEL_ID` variables in `Program.cs` with your own values.

#### Usage
1. Run the bot:
   ```bash
   dotnet run
   ```

2. Start the bot in a Telegram chat by sending the `/start` command. The bot will begin monitoring the specified YouTube channel for live streams.

#### Code Overview
The bot is implemented in C# using .NET and includes the following components:
- **Telegram Bot Client**: For interacting with the Telegram Bot API.
- **YouTube Data API Client**: For retrieving live stream data from the specified YouTube channel.
- **Serilog**: For logging bot activity to the console and a log file.

```csharp
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Telegram.Bot;
using Telegram.Bot.Exceptions;
using Telegram.Bot.Types;
using Google.Apis.Services;
using Google.Apis.YouTube.v3;
using Google.Apis.YouTube.v3.Data;
using Serilog;

class Program
{
    private static ITelegramBotClient botClient;
    private static string TELEGRAM_BOT_TOKEN = "YOUR_TELEGRAM_BOT_TOKEN";
    private static string YOUTUBE_API_KEY = "YOUR_YOUTUBE_API_KEY";
    private static string YOUTUBE_CHANNEL_ID = "YOUR_YOUTUBE_CHANNEL_ID";
    private static int CHECK_INTERVAL = 60000; // Check interval in milliseconds

    private static string chatId = null;
    private static HashSet<string> notifiedVideos = new HashSet<string>();

    static async Task Main()
    {
        Log.Logger = new LoggerConfiguration()
            .WriteTo.Console()
            .WriteTo.File("logs/log.txt", rollingInterval: RollingInterval.Day)
            .CreateLogger();

        Log.Information("Starting bot...");

        botClient = new TelegramBotClient(TELEGRAM_BOT_TOKEN);

        using var cts = new CancellationTokenSource();

        botClient.StartReceiving(
            HandleUpdateAsync,
            HandleErrorAsync,
            cancellationToken: cts.Token
        );

        var me = await botClient.GetMeAsync();
        Log.Information($"Bot started. Id: {me.Id}, Name: {me.FirstName}");

        Console.WriteLine("Press any key to exit");
        Console.ReadKey();

        // Send cancellation request to stop bot
        cts.Cancel();
        Log.Information("Bot stopped.");
    }

    static async Task HandleUpdateAsync(ITelegramBotClient botClient, Update update, CancellationToken cancellationToken)
    {
        if (update.Message is not { } message)
            return;

        if (message.Text is not { } messageText)
            return;

        Log.Information($"Received message: {messageText}");

        if (messageText.ToLower() == "/start")
        {
            chatId = message.Chat.Id.ToString();
            await botClient.SendTextMessageAsync(chatId, "Привіт! Я буду повідомляти вас про нові трансляції на каналі Beyond.");
            Log.Information("Start command received. Initiating live stream checks.");
            _ = Task.Run(() => CheckLiveStreamPeriodically());
        }
    }

    static Task HandleErrorAsync(ITelegramBotClient botClient, Exception exception, CancellationToken cancellationToken)
    {
        var errorMessage = exception switch
        {
            ApiRequestException apiRequestException => $"Telegram API Error:\n[{apiRequestException.ErrorCode}]\n{apiRequestException.Message}",
            _ => exception.ToString()
        };

        Log.Error(errorMessage);
        return Task.CompletedTask;
    }

    static async Task CheckLiveStreamPeriodically()
    {
        while (true)
        {
            if (chatId != null)
            {
                var liveStreams = await CheckLiveStream();
                if (liveStreams != null)
                {
                    foreach (var stream in liveStreams.Items)
                    {
                        var videoId = stream.Id.VideoId;
                        if (!notifiedVideos.Contains(videoId))
                        {
                            var videoUrl = $"https://www.youtube.com/watch?v={videoId}";
                            var videoDescription = await GetVideoDescription(videoId);
                            var message = $"🔴 Пряма трансляція почалася: {videoUrl}\n\n{videoDescription}";
                            await botClient.SendTextMessageAsync(chatId, message);
                            Log.Information($"Sent notification for live stream: {videoUrl}");
                            notifiedVideos.Add(videoId);

                            // Launch a timer for 5 minutes to send a follow-up message
                            _ = Task.Run(async () =>
                            {
                                await Task.Delay(300000); // 5 minute delay (300000 ms)
                                var followUpMessage = $"Ми вже поринаємо в глибини приватності, ти з нами? {videoUrl}";
                                await botClient.SendTextMessageAsync(chatId, followUpMessage);
                                Log.Information($"Sent follow-up message for live stream: {videoUrl}");
                            });
                        }
                    }
                }
            }
            await Task.Delay(CHECK_INTERVAL);
        }
    }

    static async Task<SearchListResponse> CheckLiveStream()
    {
        var youtubeService = new YouTubeService(new BaseClientService.Initializer()
        {
            ApiKey = YOUTUBE_API_KEY,
            ApplicationName = "YouTube Live Stream Checker"
        });

        var searchListRequest = youtubeService.Search.List("snippet");
        searchListRequest.ChannelId = YOUTUBE_CHANNEL_ID;
        searchListRequest.Type = "video";
        searchListRequest.EventType = SearchResource.ListRequest.EventTypeEnum.Live;

        return await searchListRequest.ExecuteAsync();
    }

    static async Task<string> GetVideoDescription(string videoId)
    {
        var youtubeService = new YouTubeService(new BaseClientService.Initializer()
        {
            ApiKey = YOUTUBE_API_KEY,
            ApplicationName = "YouTube Live Stream Checker"
        });

        var videoListRequest = youtubeService.Videos.List("snippet");
        videoListRequest.Id = videoId;

        var response = await videoListRequest.ExecuteAsync();
        if (response.Items.Count > 0)
        {
            var description = response.Items[0].Snippet.Description;
            return description.Split('\n')[0]; // First paragraph of the description
        }
        return "";
    }
}
```

### Contributing
Contributions are welcome! Please feel free to submit a Pull Request.

### License
This project is licensed under the MIT License.

### Contact
If you have any questions or suggestions, feel free to open an issue or contact me directly.

---
