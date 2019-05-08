# Quickstart: Personalize content using C# 

Display personalized content in this C# quickstart with the Personalizer service.

This sample demonstrates how to use the Personalization client library for C# to perform the following actions: 

 * Rank a list of actions for personalization.
 * Report reward to allocate to the top ranked action based on user selection for the specified event.

Getting started with Personalization involves the following steps:

1. Referencing the SDK 
1. Writing code to rank the actions you want to show to your users,
1. Writing code to send rewards to train the loop.

## Prerequisites

* You need a subscription key and token issuing service url. These are already created and available in the [Azure portal](https://portal.azure.com/#@CloudLabsAIoutlook.onmicrosoft.com/resource/subscriptions/4e09b5ee-747c-4bc6-b0d6-37550536c1a6/resourcegroups/azure-ai-64053/providers/Microsoft.CognitiveServices/accounts/personalizer/overview)
* [Visual Studio 2015 or 2017](https://visualstudio.microsoft.com/downloads/). 
    * The resource regions is `westus2`.
    * The resource key is on the Keys page.
* The Microsoft.Azure.CognitiveServices.Personalization SDK NuGet package. Installation instructions are provided below.

## Creating a new console app and referencing the Personalizer SDK 

<!--
Get the latest code as a Visual Studio solution from [GitHub] (add link).
-->

1. Create a new Visual C# Console App in Visual Studio.
1. Install the Personalization client library NuGet package. On the menu, select **Tools**, select **Nuget package Manager**, then **Manage NuGet Packages for Solution**.
1. In the **top right** of the package manager, change Package Source to **Personalizer Package**.
1. Select the **Browse** tab.
1. Check the **Include prerelease** checkbox.
1. Select **Microsoft.Azure.CognitiveServices.Personalization** when it displays.
1. Select the checkbox next to your project name, and select **Install**.

## Add the code and put in your Personalizer and Azure keys

1. Replace Program.cs with the following code. 
1. Replace `serviceKey` value with one of your Personalizer keys from the **Keys** page.
1. Replace `serviceEndpoint` with your service endpoint from the **Overview** page. An example is `https://westus2.api.cognitive.microsoft.com/`.
1. Run the program.

## Change the model update frequency 

The default model update frequency is 5 minutes. In the Azure portal, in the **Personalizer** resource, on the **Settings** page, change the rate to 1 minute for this quickstart. This frequency allows you to see the personalizer service react to the Reward API call that sends feedback to Personalizer about the user's choice. 

## Add code to rank the actions you want to show to your users

The following C# code is a complete listing to pass user information, _features, and information about your content, _actions_, to Personalizer using the SDK. Personalizer returns the top ranked action to show your user.  

```csharp
using Microsoft.Azure.CognitiveServices.Personalizer;
using Microsoft.Azure.CognitiveServices.Personalizer.Models;
using System;
using System.Collections.Generic;
using System.Linq;

namespace PersonalizerExample
{
    class Program
    {
        // The key specific to your personalizer service instance; e.g. "0123456789abcdef0123456789ABCDEF"
        private const string ApiKey = "";

        // The endpoint specific to your personalizer service instance; e.g. https://westus2.api.cognitive.microsoft.com/
        private const string ServiceEndpoint = "";

        static void Main(string[] args)
        {
            int iteration = 1;
            bool runLoop = true;

            // Get the actions list to choose from personalizer with their features.
            IList<RankableAction> actions = GetActions();

            // Initialize Personalizer client.
            PersonalizerClient client = InitializePersonalizerClient(ServiceEndpoint);

            do
            {
                Console.WriteLine("\nIteration: " + iteration++);

                // Get context information from the user.
                string timeOfDayFeature = GetUsersTimeOfDay();
                string tasteFeature = GetUsersTastePreference();

                // Create current context from user specified data.
                IList<object> currentContext = new List<object>() {
                    new { time = timeOfDayFeature },
                    new { taste = tasteFeature }
                };

                // Exclude an action for personalizer ranking. This action will be held at its current position.
                IList<string> excludeActions = new List<string> { "juice" };

                // Generate an ID to associate with the request.
                string eventId = Guid.NewGuid().ToString();

                // Rank the actions
                var request = new RankRequest(actions, currentContext, excludeActions, eventId);
                RankResponse response = client.Rank(request);

                Console.WriteLine("\nPersonalizer service thinks you would like to have: " + response.RewardActionId + ". Is this correct? (y/n)");

                float reward = 0.0f;
                string answer = GetKey();

                if (answer == "Y")
                {
                    reward = 1;
                    Console.WriteLine("\nGreat! Enjoy your food.");
                }
                else if (answer == "N")
                {
                    reward = 0;
                    Console.WriteLine("\nYou didn't like the recommended food choice.");
                }
                else
                {
                    Console.WriteLine("\nEntered choice is invalid. Service assumes that you didn't like the recommended food choice.");
                }

                Console.WriteLine("\nPersonalizer service ranked the actions with the probabilities as below:");
                foreach (var rankedResponse in response.Ranking)
                {
                    Console.WriteLine(rankedResponse.Id + " " + rankedResponse.Probability);
                }

                // Send the reward for the action based on user response.
                client.Reward(response.EventId, new RewardRequest(reward));

                Console.WriteLine("\nPress q to break, any other key to continue:");
                runLoop = !(GetKey() == "Q");

            } while (runLoop);
        }

        /// <summary>
        /// Initializes the personalizer client.
        /// </summary>
        /// <param name="url">Azure endpoint</param>
        /// <returns>Personalizer client instance</returns>
        static PersonalizerClient InitializePersonalizerClient(string url)
        {
            PersonalizerClient client = new PersonalizerClient(
                new ApiKeyServiceClientCredentials(ApiKey)) {Endpoint = url};

            return client;
        }

        /// <summary>
        /// Get users time of the day context.
        /// </summary>
        /// <returns>Time of day feature selected by the user.</returns>
        static string GetUsersTimeOfDay()
        {
            string[] timeOfDayFeatures = new string[] { "morning", "afternoon", "evening", "night" };

            Console.WriteLine("\nWhat time of day is it (enter number)? 1. morning 2. afternoon 3. evening 4. night");
            if (!int.TryParse(GetKey(), out int timeIndex) || timeIndex < 1 || timeIndex > timeOfDayFeatures.Length)
            {
                Console.WriteLine("\nEntered value is invalid. Setting feature value to " + timeOfDayFeatures[0] + ".");
                timeIndex = 1;
            }

            return timeOfDayFeatures[timeIndex - 1];
        }

        /// <summary>
        /// Gets user food preference.
        /// </summary>
        /// <returns>Food taste feature selected by the user.</returns>
        static string GetUsersTastePreference()
        {
            string[] tasteFeatures = new string[] { "salty", "sweet" };

            Console.WriteLine("\nWhat type of food would you prefer (enter number)? 1. salty 2. sweet");
            if (!int.TryParse(GetKey(), out int tasteIndex) || tasteIndex < 1 || tasteIndex > tasteFeatures.Length)
            {
                Console.WriteLine("\nEntered value is invalid. Setting feature value to " + tasteFeatures[0] + ".");
                tasteIndex = 1;
            }

            return tasteFeatures[tasteIndex - 1];
        }

        /// <summary>
        /// Creates personalizer actions feature list.
        /// </summary>
        /// <returns>List of actions for personalizer.</returns>
        static IList<RankableAction> GetActions()
        {
            IList<RankableAction> actions = new List<RankableAction>
            {
                new RankableAction
                {
                    Id = "pasta",
                    Features =
                    new List<object>() { new { taste = "salty", spiceLevel = "medium" }, new { nutritionLevel = 5, cuisine = "italian" } }
                },

                new RankableAction
                {
                    Id = "ice cream",
                    Features =
                    new List<object>() { new { taste = "sweet", spiceLevel = "none" }, new { nutritionalLevel = 2 } }
                },

                new RankableAction
                {
                    Id = "juice",
                    Features =
                    new List<object>() { new { taste = "sweet", spiceLevel = "none" }, new { nutritionLevel = 5 }, new { drink = true } }
                },

                new RankableAction
                {
                    Id = "salad",
                    Features =
                    new List<object>() { new { taste = "salty", spiceLevel = "low" }, new { nutritionLevel = 8 } }
                }
            };

            return actions;
        }

        private static string GetKey()
        {
            return Console.ReadKey().Key.ToString().Last().ToString().ToUpper();
        }
    }
}
```

## Run the program

Build and run the program. The quickstart program asks a couple of questions to gather user preferences, known as features, then provides the top action.

![The quickstart program asks a couple of questions to gather user preferences, known as features, then provides the top action.](media/csharp-quickstart-commandline-feedback-loop/quickstart-program-feedback-loop-example.png)

## Clean up resources
When you are done with the quickstart, remove all the files created in this quickstart. 

## Next steps

[How Personalizer works](how-personalizer-works.md)


