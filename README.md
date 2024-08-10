
```markdown
# YouTube Live Stream Notifier Bot

This Telegram bot automatically notifies users about new live streams on a specified YouTube channel.

## Features

- **Live Stream Checking:** The bot periodically checks for new live streams on the specified YouTube channel.
- **Telegram Notifications:** If a new live stream is found, the bot sends a message in a Telegram chat with a link to the stream and a short description.
- **Logging:** Logging is handled via `Serilog`. Logs are stored in the `logs/log.txt` file.

## Technologies Used

- **C# .NET:** The main programming language for the bot.
- **Telegram.Bot API:** For working with Telegram.
- **YouTube Data API v3:** For checking live streams on YouTube.
- **Serilog:** For logging events.

## Configuration

Before running the bot, you need to set up the following parameters:

- `TELEGRAM_BOT_TOKEN`: Your Telegram bot token.
- `YOUTUBE_API_KEY`: API key for accessing YouTube Data API v3.
- `YOUTUBE_CHANNEL_ID`: The ID of the YouTube channel to check for live streams.
- `CHECK_INTERVAL`: Interval for checking for new live streams (in milliseconds).

### Example Configuration

```csharp
private static string TELEGRAM_BOT_TOKEN = "YOUR_TELEGRAM_BOT_TOKEN";
private static string YOUTUBE_API_KEY = "YOUR_YOUTUBE_API_KEY";
private static string YOUTUBE_CHANNEL_ID = "YOUR_YOUTUBE_CHANNEL_ID";
private static int CHECK_INTERVAL = 60000; // Interval in milliseconds
```

## How to Run

1. Install all necessary dependencies for the project.
2. Set up the parameters mentioned above.
3. Run the project using the `dotnet run` command or the appropriate method in your IDE.

## Usage

- Start the bot, then send the `/start` command in your Telegram chat.
- The bot will begin checking for new live streams and will send notifications when they are found.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
```

This `README.md` provides all the necessary information for understanding the purpose of the project, its setup, and its usage. It also includes basic licensing information.
