## How to get AWS access keys

_Originally published on Sep 1, 2023 [here](https://medium.com/@jannden/how-to-get-aws-access-keys-81cad0366418)._

---

This is a step-by-step guide on getting the AWS access keys so that you can use Amazon APIs in your project. So let's get AWS security credentials for your project now.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*ktaUiZd3zLsqG6d12aO2Uw.jpeg)

### Get AWS security credentials (access keys)

Sign in to the AWS Management Console and open the IAM console at <https://console.aws.amazon.com/iam/>

In the navigation pane, choose Users under Access management. Then click on Add users.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*ztvSpftA7xPEMRL8BtDSOg.png)

Choose the user name. It's just for your reference. Then click on the Next button.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*cVUC2JiIpxJCYKJ7GffVOw.png)

Choose the Attach policies directly option.

For example, if we were building [a transcription app with translation and text-to-voice capabilities](https://medium.com/@jannden/3d020ec2f729), we would need access to three AWS services (you select your policies accordingly to your needs). So for our example case, we would filter the policies by `transcribe`, `Polly`, and `translate` keywords (set the connector to OR instead of AND to see results for all three keywords at once).

In the results area below the search, mark these policies:

- AmazonPollyFullAccess
- TranslateFullAccess
- AmazonTranscribeFullAccess

This is how it would look:

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*EmjG1FXyO3gqJZM0jlZtQg.png)

Confirm your selection at the bottom of the page and proceed to the review step. Nothing to do here, just confirm again by clicking on the Create user button.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*RxLEInExDknBbft-_HAWxQ.png)

Once the user is created, click on its name. In my case, the user name is MyApp.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*muY2yYSYS6lj6QC0tgJb7A.png)

On the user page, switch to the Security credentials tab. Scroll down to find a button named Create access key and click on it.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*p6TNEAgcxa3VtfMIUDJUOw.png)

Choose an option that is relevant to your project. For this [tutorial series](https://medium.com/@jannden/3d020ec2f729) (building a transcription app), we will choose the option Application running outside of AWS. Confirm at the bottom of the page and proceed to the next step.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*xKFQrwgJ00C4hoKge2HHqA.png)

Setting a description tag is optional, we can just skip it and click directly on the Create access key button.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*Ki8XWg5QRjnKH_A_zua8cg.png)

And finally, you are provided with the access keys.

Note down both:

- Access key
- Secret access key

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*jKKsgCbmNpREK2KUQqAK-w.png)

Finally, for using AWS APIs, you will also need to specify a region. You will find a table listing all available regions in [the official documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html). For the AWS APIs, you will need to pick one region from the column Region, for example: `eu-central-1`.

### Conclusion

This short article showed how to create AWS access keys so that you can use these security credentials to access Amazon APIs.

If you found it helpful, please click the clap 👏 button. Also, feel free to comment! I'd be happy to help :)
