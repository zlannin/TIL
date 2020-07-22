# Mocking Internal Constructors

The other day I was implementing a feature to send push notifications to Firebase Cloud Messaging (FCM). The library was very easy to use, except one part, the testing. I created an interface to wrap around `SendMulticastAsync` so I can mock. When writing my unit tests to my surprise I could not initialize `BatchResponse` because it does not contain a constructor that takes 0 arguments. I open JetBrains dotPeek so I can view the decompiled DLL where I notice this.

```
  public sealed class BatchResponse
  {
    internal BatchResponse(IEnumerable<SendResponse> responses)
    {
      ...
    }
    ...
    }
  }
```

This is the first time I have seen in internal constructor. One way to initialize this class was using reflection to `CreateInstance` with `BindingFlags`.

 ```
public static T CreateInstance<T>(params object[] args)
{
    var type = typeof(T);
    var instance = type.Assembly.CreateInstance(
        type.FullName, false,
        BindingFlags.Instance | BindingFlags.NonPublic,
        null, args, null, null);
    return (T)instance;
}
```

With that method I and a few helper methods I can easily write a unit test for sending a push notification to FCM.

```
[TestMethod]
public async Task SendPushNotification() {
    var args = new SendResponse[]
    {
        CreateInvalidTokenResponse()
    }.AsEnumerable();
    var mockBatchResponse = CreateInstance<BatchResponse>(args);

    mockFCMClient
        .Setup(m => m.SendMulticastMessage(It.IsAny<MulticastMessage>()))
        .Returns(Task.FromResult(mockBatchResponse));

    ...
}

private SendResponse CreateSuccessfulResponse(string messageId = null)
{
    messageId = messageId ?? Guid.NewGuid().ToString();
    return CreateInstance<SendResponse>(messageId);
}

private SendResponse CreateInvalidTokenResponse()
{
    var firebaseEx = CreateFirebaseException(ErrorCode.InvalidArgument, "The registration token is not a valid FCM registration token", MessagingErrorCode.InvalidArgument);
    return CreateInstance<SendResponse>(firebaseEx);
}

private FirebaseMessagingException CreateFirebaseException(ErrorCode errorCode, string message, MessagingErrorCode? fcmCode = null, Exception inner = null, HttpResponseMessage response = null)
{
    return CreateInstance<FirebaseMessagingException>(errorCode, message, fcmCode, inner, response);
}
```