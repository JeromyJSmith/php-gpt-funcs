# PHP ChatGPT-4 Library with Function Calling

This is a simple library I have created for interacting with the ChatGPT-4 API with function calling.

**This is a work in progress**

## Basic usage

Send a message and get response back:
```php
$chatgpt = new ChatGPT( "YOUR_API_KEY" );
$chatgpt->umessage( "Write a short story about a man named Mike" );

// Prints out the story
echo $chatgpt->response()->content;
```

## System message

You can set a system message to make ChatGPT behave in a specific manner.

```php
$chatgpt = new ChatGPT( "YOUR_API_KEY" );
$chatgpt->smessage( "You are a chatbot that only answers with riddles" );
$chatgpt->umessage( "What is the distance from the earth to the moon?" );

// Without numbers, I offer you a clue,
// As shadow at peak, or notch in night's hue.
// To calculate this gap is no simple boon,
// What's termed 'a round trip' by the light of the moon.
echo $chatgpt->response()->content;
```

## Function calling

You can pass PHP functions to the ChatGPT class with the `add_function` method. The function and parameter descriptions will be extracted automatically from the DocBlock comment. Parameter types will be extracted automatically from the function with `ReflectionFunction`.

```php
/**
 * Gets the current weather information
 * @param string $location The location for which to get the weather
 */
function get_current_weather( string $location ) {
    if( $location === "California" ) {
        return "It's nice and sunny";
    } else {
        return "It's cold and windy";
    }
}

$chatgpt = new ChatGPT( "YOUR_API_KEY" );
$chatgpt->add_function( "get_current_weather" );

$chatgpt->umessage( "What's the weather like in California?" );
// It's nice and sunny in California
echo $chatgpt->response()->content . PHP_EOL;

$chatgpt->umessage( "What's the weather like in Alaska?" );
// It's cold and windy in Alaska
echo $chatgpt->response()->content . PHP_EOL;
```

## Saving chat history

You can save the chat history to a file and load it from a file using the `savefunction` and `loadfunction` methods. Pass in your own function that handles the loading / saving in your preferred way.

```php
$chatgpt = new ChatGPT( "YOUR_API_KEY" );
$chatgpt->savefunction( function( $message, $chat_id ) {
    file_put_contents(
        "chat_" . $chat_id . ".txt",
        json_encode( $message ) . PHP_EOL,
        FILE_APPEND
    );
} );
$chatgpt->loadfunction( function( $chat_id ) {
    $messages = @file( "chat_" . $chat_id . ".txt" );
    return array_map( function( $message ) {
        return json_decode( $message );
    }, $messages ?: [] );
} );
$chatgpt->load();

$messages = $chatgpt->messages();
$message_count = count( $messages );

// Prints out message objects
foreach( $messages as $message ) {
    print_r( $message );
}

$chatgpt->umessage( "What is " . ( ( $message_count / 2 ) + 1 ) . " * 5?" );
// Prints 5, 10, 15, 20, 25, etc. on consecutive runs
echo $chatgpt->response()->content . PHP_EOL;
```
