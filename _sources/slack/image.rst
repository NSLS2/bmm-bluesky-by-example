
Posting an image to Slack
=========================

The user experience at the beamline benefits a lot by having text
messages regularly posted to Slack.  Adding images makes that
experience much richer.  Being able to see the result of a measurement
on Slack makes it much easier to walk away from the beamline and have
confidence that things are proceeding apace.

Obtain an uploader token
------------------------

The process to enabling image posts is rather similar to posting text.
Here is a handy page that I followed when I was setting this up:
https://hamzaafridi.com/sending-a-file-to-a-slack-channel-using-api/

The interface is similar in that you need to establish a App and
associate with a specific channel on a specific workspace.  But then
you need to use the Slack python interface to post a file.  There is
likely a way to stream an image directly to Slack, but I have
implemented it as posting a file.  The images I post to Slack are
things I also write to disk and include in the data product that the
user takes home from the beamline.  Since I am writing files in any
case, it seemed sensible to implement this as a file posting.


Code for posting an image
-------------------------

Setting up the App results in being given a token for uploading
files.  In lines 4 to 10 of the code below, I am reading this token
from the beamline NAS.  This token is also a thing that must not be
committed to the beamline Github repo.  Again, keeping it on a NAS or
on Lustre, while not actually secure, is adequate.

Here is the code for posting an image stored in a file to a Slack
channel.

.. code-block:: python
   :linenos:

      ## Simple but useful guide to configuring a slack app:        
      ## https://hamzaafridi.com/sending-a-file-to-a-slack-channel-using-api/
      def img_to_slack(imagefile):
          token_file = os.path.join(startup_dir, 'BMM', 'image_uploader_token')
          try:
              with open(token_file, "r") as f:
                  token = f.read().replace('\n','')
          except:
              post_to_slack(f'failed to post image: {imagefile}')
              return()
          client = WebClient(token=token)
          #client = WebClient(token=os.environ['SLACK_API_TOKEN'])
          try:
              response = client.files_upload(channels='#beamtime', file=imagefile)
              assert response["file"]  # the uploaded file
          except SlackApiError as e:
              post_to_slack('failed to post image: {imagefile}')
              # You will get a SlackApiError if "ok" is False
              assert e.response["ok"] is False
              assert e.response["error"]  # str like 'invalid_auth', 'channel_not_found'
              print(f"Got an error: {e.response['error']}")
          except Exception as em:
              print("EXCEPTION: " + str(em))
              print(f'failed to post image: {imagefile}')
	      post_to_slack(f'failed to post image: {imagefile}')

This can be used at the ``bsui`` command line or in a Bluesky plan
like so:

.. code-block:: python

   from beamline_slack import img_to_slack
   img_to_slack('/path/to/data/plot.png')

The file posted does not need to be a PNG image.  You can post all
kinds of files in this manner.

Here is what it looks like for BMM.  At the end of a sequence of
repetitions on the same sample, the data are merged and lightly
reduced.  Matplotlib is used to make a fun representation of the
reduced data, which is saved to a PNG file.  That PNG file is then
posted to the Slack channel.


.. _fig-slack-image:
.. figure:: ../_static/slack_image.png
   :target: ../_static/slack_image.png
   :align: center

   Posting images to Slack.
