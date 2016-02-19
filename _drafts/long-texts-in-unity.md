----
 -layout: post
 -title: "Long texts in Unity"
 -description: ""
 -category: 
 -tags: []
 ----

 TL;DR:

 * Unity UI breaks on long texts
 * Cut the text into paragraphs
 * Dynamically create and destroy text elements for each paragraph
 
Test project: https://gitlab.com/golergka/unity-long-text

# The problem

Let's start with discovering the problem: trying to put a giant, enormous wall of text into Unity UI in the most naive way possible. (You will find the related files in `Naive` folder in the linked project.) Here, I created the scroll parent, scroll view, and finally the text element itself, with `ContentSizeFitter` component which will automatically adjust it's height. In earlier versions of Unity UI, I could've placed the text directly under the scroll rect itself, but now scroll rect acts as a layout group, so I have to use explicit scroll view game object in the hierarchy.

Now, all I have to do is to put in the huge wall of texts. With basic Lorem Ipsum generator, getting the text is easy. However, instead of putting it directly into the text element, I want to save it to disc as an plain old text dile and load it dynamically:

```csharp
using UnityEngine;
using UnityEngine.UI;

[RequireComponent(typeof(Text))]
public class NaiveApproach : MonoBehaviour
{
	[SerializeField] private TextAsset	mr_WallOfText;

	private void Start()
	{
		GetComponent<Text>().text = mr_WallOfText.text;
	}
}
```

And here's the whole thing set up:

![naive approach]({{ site.url }}/assets/long-texts-in-unity/naitve-set-up.png)

Looks alright? Not really. When you try to start it, text gets cut up, the whole Unity editor starts to run slow as hell, and you get this in the console:

![native-errors]({{ site.url }}/assets/long-texts-in-unity/native-errors.png)

The mistakes itself may depend on your particular version of Unity. In 5.1, the Canvas itself would cry that it's out of the 65535 vertices it can use, in 5.2 the TextMeshGenerator knows to cut the text himself. But either way, you're in trouble.

# Cut the text

First thing we're going to try is to cut the text into cute little chunks so that each Text Unity element has to eat up only a little bit. Now, instead of putting the script on the text element, we put it on the special container that we will be creating all the text elements in. Here's the script — it should be pretty straightforward.

```csharp
using UnityEngine;
using UnityEngine.UI;

public class CutText : MonoBehaviour
{
	[SerializeField] private TextAsset		mr_WallOfText;
	[SerializeField] private Text			mr_ParagraphPrefab;

	private void Start()
	{
		var wallOfText = mr_WallOfText.text;

		/* Be carefull with what symbol are you splitting the text in: it's \n in Unix systems and
		 * \n\r in Windows. System.Envrionment.Newline might sound like a good idea, but it's not,
		 * because it changes dynamically depending on what platform are you running the code, and
		 * instead, you want it to be tied to the platform on which you edited the text.
		 *
		 * In my case, I know it's \n.
		 */
		var paragraphs = wallOfText.Split('\n');
		foreach(var p in paragraphs)
		{
			var pText = Instantiate(mr_ParagraphPrefab) as Text;
			pText.transform.SetParent(transform, false);
			pText.text = p;
		}
	}
}
```

Now, all is left is to set up container and prefab themselves. The container needs to hold multiple UI elements one after another, and Vertical Layout Group component is created just for that. And we also want it to change the size to fit all the children in, so we add Content Size Fitter as well. Here's the full container configuration:

![container-configuration]({{ site.url }}/assets/long-texts-in-unity/container-configuration.png)
