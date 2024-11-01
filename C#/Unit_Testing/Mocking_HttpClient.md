# Mocking HttpClient  

**NB! The proper way of doing this is to not mock HttpClient.**
- **Make an interface class. This has an instance of HttpClient in it.**
- **Make a wrap method around the call methods in HttpClient. HttpClient has Get, Post, Put, Patch and Delete methods. Make a wrap method for each enpoint call you make.**
- **Return the HttpResponseMessage from the wrap method.**
- **All logic is in the class that call the interface method.**
- **Mock the interface.**
- **Interface A : Class A contains HttpClient. Class A implements methods for each for each API call you make. Mock Class A and the methods there. You will not need to touch HttpClient.**
```
public class ApiCaller : IApiCaller
{
    private readonly Httpclient _client;

    public ApiCaller(HttpClient client)
    {
        _client = client;
    }

    public async Task<HttpResponseMessage> CallApiPoint1()
    {
        return _client.GetAsync("URL");
    }
}
```

### Test Example with nSubstitute
```
[Fact]
public async void ShouldCallCreatePostApi()
{
    //Arrange
    //Creates a mock of the HttpMesssageHandler class

    var apiCaller = Substitute.For<IApiCaller>

    //This is a dummy of the response a real API call would generate. This needs to be populated ONLY with what will be used by the logic in the method being tested.
    var response = new HttpResponseMessage
    {
        StatusCode = HttpStatusCode.OK,
        Content = new StringContent(@"{ ""id"": 101 }"),
    };

    //Sets the return value for the mocked method.

    apiCaller.CallApiPoint1(default).ReturnsForAnyArgs(Task<response>);

    //Remember that when using mocked objects you need to use the Object belonging to the mock.    

    var returnHandler = new ReturnHandler(apiCaller);
            
    //Act
    var result = await returnHandler.GetInfo("URL");

    //Assert
    result.StatusCode.Should().Be(HttpStatusCode.OK);
}
```  

## Obsolete

Methods in HttpClient are not virtual and not set up for mocking. This makes it somewhat of a problem when attempting to mock HttpClient.  
Setting up an Interface does not solve this issue.  
  
  The solution to this issue lies in HttpMessageHandler, as this class can be mocked. HttpClient invokes HttpMessageHandler methods in such a way that mocking the **SendAsync** (as an example) method in HttpMessageHandler allows for the use of HttpClient in unit tests.  
    
I found this explanation helpful when setting this up for the first time:
 - Resource - [How to test HttpClient with Moq](https://dev.to/gautemeekolsen/how-to-test-httpclient-with-moq-in-c-2ldp)  


### Code example
```
[Fact]
public async void ShouldCallCreatePostApi()
{
    //Arrange
    //Creates a mock of the HttpMesssageHandler class
    var handlerMock = new Mock<HttpMessageHandler>();

    //This is a dummy of the response a real API call would generate. This needs to be populated ONLY with what will be used by the logic in the method being tested.
    var response = new HttpResponseMessage
    {
        StatusCode = HttpStatusCode.OK,
        Content = new StringContent(@"{ ""id"": 101 }"),
    };

    //Sets the mocked behaviour. Here that when SendAsync is called with this override, it will return the the specified response. This overrides the normal behaviour of the method, makes it skip running the internal code and just returns the predetermined value.
    handlerMock
        .Protected()
        .Setup<Task<HttpResponseMessage>>(
            "SendAsync",
            ItExpr.IsAny<HttpRequestMessage>(),
            ItExpr.IsAny<CancellationToken>())
        .ReturnsAsync(response);

    //Remember that when using mocked objects you need to use the Object belonging to the mock.    
    var httpClient = new HttpClient(handlerMock.Object);
    var posts = new Posts(httpClient);
            
    //Act
    var retrievedPosts = await posts.CreatePost("Best post");

    //Assert
    handlerMock.Protected().Verify(
        "SendAsync",
        Times.Exactly(1),
        ItExpr.Is<HttpRequestMessage>(req => req.Method == HttpMethod.Post),
        ItExpr.IsAny<CancellationToken>());
}
```  
  
**ItExpr** enables an evaluation of the input. In this example the method is only mocked if the **HttpMessage** has the **URI** specified in the evaluating expression.
```
        handlerMock
            .Protected()
            .SetupSequence<Task<HttpResponseMessage>>(
                "SendAsync",
                ItExpr.Is<HttpRequestMessage>( x => x.RequestUri == new Uri("https://apiendpoint.com/post")),
                ItExpr.IsAny<CancellationToken>())
            .ReturnsAsync(response)
            .ReturnsAsync(response)
            .ReturnsAsync(response)
            .ReturnsAsync(responseError);
```