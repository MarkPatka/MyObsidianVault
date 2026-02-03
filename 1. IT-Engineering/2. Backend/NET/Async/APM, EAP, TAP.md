

| Pattern        | Asynchronous Programming Model                                                                                                                    | Event-based Asynchronous Pattern                                                                                               | Task-based Asynchronous Pattern                                                                                                                                                                        |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Introduced     | .NET 1.0                                                                                                                                          | .NET 2.0                                                                                                                       | .NET 4.0+                                                                                                                                                                                              |
| Return Type    | IAsyncResult                                                                                                                                      | Events and Callbacks                                                                                                           | Task, Task<>                                                                                                                                                                                           |
| Usage          | BeginX/EndX methods                                                                                                                               | MethodAsync & events                                                                                                           | async/await                                                                                                                                                                                            |
| Cancellation   | Manual via IAsyncResult                                                                                                                           | CancelAsync method                                                                                                             | CancellationToken                                                                                                                                                                                      |
| Pros           | • Universal in older .NET Framework  <br>• Low-level control                                                                                      | • Better than APM for UI scenarios  <br>• Progress and cancellation support  <br>• Multiple operations per object              | • Modern, clean syntax  <br>• Compiler support with async/await  <br>• Excellent composability  <br>• Built-in error handling  <br>• Unified cancellation model  <br>• Easy to understand and maintain |
| Cons           | • Complex callback management  <br>• Difficult error handling  <br>• Poor composability  <br>• Callback hell risk  <br>• No built-in cancellation | • Event handler overhead  <br>• Limited composability  <br>• Inconsistent implementations  <br>• UI thread marshaling required | • Requires .NET 4.0+  <br>• Learning curve for beginners  <br>• Can hide complexity                                                                                                                    |
| Current Status | Legacy, avoid in new code                                                                                                                         | Legacy, superseded by TAP                                                                                                      | **Recommended** for all new development                                                                                                                                                                |
## APM (Asynchronous Programming Model) - BeginRead/EndRead

```csharp
public class FileReaderAPM
    {
        private FileStream _fileStream;
        private byte[] _buffer;

        public FileReaderAPM(string filePath, int bufferSize = 4096)
        {
            _fileStream = new FileStream(filePath, FileMode.Open, FileAccess.Read, FileShare.Read, bufferSize, true);
            _buffer = new byte[bufferSize];
        }

        public void ReadFileAsync(Action<byte[], int> onComplete, Action<Exception> onError)
        {
            _fileStream.BeginRead(_buffer, 0, _buffer.Length, asyncResult =>
            {
                try
                {
                    int bytesRead = _fileStream.EndRead(asyncResult);
                    onComplete?.Invoke(_buffer, bytesRead);
                }
                catch (Exception ex)
                {
                    onError?.Invoke(ex);
                }
            }, null);
        }

        public void Close()
        {
            _fileStream?.Close();
            _fileStream?.Dispose();
        }
    }

    // Пример использования APM
    public class APMExample
    {
        public static void Demo()
        {
            Console.WriteLine("=== APM Pattern Demo ===");
            
            var reader = new FileReaderAPM("example.txt");
            
            reader.ReadFileAsync(
                onComplete: (buffer, bytesRead) =>
                {
                    if (bytesRead > 0)
                    {
                        string content = Encoding.UTF8.GetString(buffer, 0, bytesRead);
                        Console.WriteLine($"Read {bytesRead} bytes:");
                        Console.WriteLine(content);
                    }
                    else
                    {
                        Console.WriteLine("End of file reached");
                    }
                    reader.Close();
                },
                onError: (ex) =>
                {
                    Console.WriteLine($"Error: {ex.Message}");
                    reader.Close();
                }
            );
            
            Console.WriteLine("Read operation started...");
        }
    }
```

## EAP (Event-based Asynchronous Pattern) - WebClient

```csharp
    public class FileDownloaderEAP
    {
        private WebClient _webClient;

        public FileDownloaderEAP()
        {
            _webClient = new WebClient();
        }

        public void DownloadFileAsync(string url, string destinationPath)
        {
            // Подписываемся на событие завершения
            _webClient.DownloadFileCompleted += OnDownloadFileCompleted;
            
            // Подписываемся на событие прогресса (опционально)
            _webClient.DownloadProgressChanged += OnDownloadProgressChanged;

            Console.WriteLine($"Starting download from: {url}");
            _webClient.DownloadFileAsync(new Uri(url), destinationPath);
        }

        private void OnDownloadFileCompleted(object sender, System.ComponentModel.AsyncCompletedEventArgs e)
        {
            if (e.Error != null)
            {
                Console.WriteLine($"Download failed: {e.Error.Message}");
            }
            else if (e.Cancelled)
            {
                Console.WriteLine("Download was cancelled");
            }
            else
            {
                Console.WriteLine("Download completed successfully!");
            }

            // Отписываемся от событий
            _webClient.DownloadFileCompleted -= OnDownloadFileCompleted;
            _webClient.DownloadProgressChanged -= OnDownloadProgressChanged;
            _webClient.Dispose();
        }

        private void OnDownloadProgressChanged(object sender, DownloadProgressChangedEventArgs e)
        {
            Console.WriteLine($"Progress: {e.ProgressPercentage}% ({e.BytesReceived}/{e.TotalBytesToReceive} bytes)");
        }
    }

    // Пример использования EAP
    public class EAPExample
    {
        public static void Demo()
        {
            Console.WriteLine("\n=== EAP Pattern Demo ===");
            
            var downloader = new FileDownloaderEAP();
            downloader.DownloadFileAsync(
                "https://example.com/sample.txt",
                "downloaded_file.txt"
            );
            
            Console.WriteLine("Download initiated...");
        }
    }

```

##  TAP (Task-based Asynchronous Pattern) - async/await

```csharp
public class HtmlParserTAP
    {
        private readonly HttpClient _httpClient;

        public HtmlParserTAP()
        {
            _httpClient = new HttpClient();
        }

        public async Task<string> FetchAndParseHtmlAsync(string url)
        {
            try
            {
                Console.WriteLine($"Fetching HTML from: {url}");
                
                // Загружаем HTML
                string htmlContent = await _httpClient.GetStringAsync(url);
                
                Console.WriteLine($"Downloaded {htmlContent.Length} characters");
                
                // Простой "парсинг" - извлекаем заголовок
                string title = ExtractTitle(htmlContent);
                
                return title;
            }
            catch (HttpRequestException ex)
            {
                Console.WriteLine($"HTTP Error: {ex.Message}");
                throw;
            }
        }

        private string ExtractTitle(string html)
        {
            int titleStart = html.IndexOf("<title>", StringComparison.OrdinalIgnoreCase);
            int titleEnd = html.IndexOf("</title>", StringComparison.OrdinalIgnoreCase);

            if (titleStart >= 0 && titleEnd > titleStart)
            {
                titleStart += 7; // длина "<title>"
                return html.Substring(titleStart, titleEnd - titleStart).Trim();
            }

            return "Title not found";
        }

        public void Dispose()
        {
            _httpClient?.Dispose();
        }
    }

    // Пример использования TAP
    public class TAPExample
    {
        public static async Task DemoAsync()
        {
            Console.WriteLine("\n=== TAP Pattern Demo ===");
            
            using (var parser = new HtmlParserTAP())
            {
                string title = await parser.FetchAndParseHtmlAsync("https://example.com");
                Console.WriteLine($"Page title: {title}");
            }
        }
    }
```

## Поскольку APM и EAP модели - легаси, вот как они будут выглядеть переписанными в TAP
```csharp
// Пример 2 (загрузка файла) переписан на TAP
public class FileDownloaderTAP
{
	private readonly HttpClient _httpClient;

	public FileDownloaderTAP()
	{
		_httpClient = new HttpClient();
	}

	public async Task DownloadFileAsync(string url, string destinationPath)
	{
		Console.WriteLine($"Starting download from: {url}");
		
		// Загружаем содержимое
		byte[] fileBytes = await _httpClient.GetByteArrayAsync(url);
		
		// Сохраняем в файл
		await File.WriteAllBytesAsync(destinationPath, fileBytes);
		
		Console.WriteLine($"Download completed! Saved to: {destinationPath}");
	}

	// Альтернативный вариант с отслеживанием прогресса
	public async Task DownloadFileWithProgressAsync(string url, string destinationPath, IProgress<double> progress)
	{
		using (var response = await _httpClient.GetAsync(url, HttpCompletionOption.ResponseHeadersRead))
		{
			response.EnsureSuccessStatusCode();
			
			long? totalBytes = response.Content.Headers.ContentLength;
			
			using (var contentStream = await response.Content.ReadAsStreamAsync())
			using (var fileStream = new FileStream(destinationPath, FileMode.Create, FileAccess.Write, FileShare.None, 8192, true))
			{
				byte[] buffer = new byte[8192];
				long totalRead = 0;
				int bytesRead;
				
				while ((bytesRead = await contentStream.ReadAsync(buffer, 0, buffer.Length)) > 0)
				{
					await fileStream.WriteAsync(buffer, 0, bytesRead);
					totalRead += bytesRead;
					
					if (totalBytes.HasValue)
					{
						double percentage = (double)totalRead / totalBytes.Value * 100;
						progress?.Report(percentage);
					}
				}
			}
		}
		
		Console.WriteLine("Download completed!");
	}

	public void Dispose()
	{
		_httpClient?.Dispose();
	}
}


// Демонстрация TAP версий
public class TAPConversionDemo
{
	public static async Task DemoAsync()
	{
		Console.WriteLine("\n=== APM/EAP Examples Rewritten with TAP ===");
		
		// Пример 1: Чтение файла
		Console.WriteLine("\n--- File Reading (TAP) ---");
		var fileReader = new FileReaderTAP();
		try
		{
			string content = await fileReader.ReadFileAsync("example.txt");
			Console.WriteLine($"File content: {content}");
		}
		catch (Exception ex)
		{
			Console.WriteLine($"Error reading file: {ex.Message}");
		}
		
		// Пример 2: Загрузка файла
		Console.WriteLine("\n--- File Downloading (TAP) ---");
		using (var downloader = new FileDownloaderTAP())
		{
			try
			{
				await downloader.DownloadFileAsync(
					"https://example.com/sample.txt",
					"downloaded_tap.txt"
				);
			}
			catch (Exception ex)
			{
				Console.WriteLine($"Error downloading file: {ex.Message}");
			}
			
			// С отслеживанием прогресса
			Console.WriteLine("\n--- With Progress Reporting ---");
			var progress = new Progress<double>(percentage =>
			{
				Console.WriteLine($"Progress: {percentage:F2}%");
			});
			
			try
			{
				await downloader.DownloadFileWithProgressAsync(
					"https://example.com/largefile.zip",
					"large_downloaded.zip",
					progress
				);
			}
			catch (Exception ex)
			{
				Console.WriteLine($"Error: {ex.Message}");
			}
		}
	}
}
```