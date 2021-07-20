import qi


def main(session):
    dialog = session.service("ALDialog")
    dialog.setLanguage("English")
    faceDetect = session.service("ALFaceDetection")
    memory = session.service("ALMemory")
    tts = session.service("ALTextToSpeech")
    motion = session.service("ALMotion")
    tracker = session.service("ALTracker")
    userInfo = session.service("ALUserInfo")
    userSession = session.service("ALUserSession")
    tabletService = session.service("ALTabletService")
    peoplePercept = session.service("ALPeoplePerception")

    # wake up
    motion.wakeUp()

    # Tracking
    # Add target to track
    tracker.registerTarget("Face", 0.1)
    # Then, start tracker
    tracker.track("Face")

    # face detection
    # subscribe to ALFaceDetection proxy
    # this means that the module will write in ALMemory with the given period (500) below
    faceDetect.subscribe("Test_Face", 500, 0.0)

    # Dialog
    # writing topics' qi chat code as text strings (end-of-line characters are important!)
    topic_content = ('topic: ~example_topic_content()\n'
                     'language: enu\n'
                     'concept:(myname) ["my name is" "call me" "i am" "im"]\n'
                     'concept:(name) ["Clement" "Cedric" "Philippe" "Jean Philippe" "Paulo"]\n'
                     'u:(name: _*) ^setUserInfo(name, $1)\n'
                     'u:(~myname _~name) ^setUserInfo(name, $1)\n'
                     'u:(["who am i" "whats my name"]) ^first["you are ^getUserInfo(name)" "i dont know"]\n'
                     'u:(forget my name) ok ^removeUserInfo(name)\n'
                     )

    # Loading the topics directly as text strings
    topic_name = dialog.loadTopicContent(topic_content)
    # Activating the loaded topics
    dialog.activateTopic(topic_name)
    # Starting the dialog engine - we need to type an arbitrary string as the identifier
    # We subscribe only ONCE, regardless of the number of topics we have activated
    dialog.subscribe('my_dialog_example')

    got_face = False
    prev_name = ""

    try:
        while True:
            # display logo on tablet
            tabletService.setBackgroundColor("#FFFFFF")
            tabletService.showImage("http://198.18.0.1/apps/testing-f6fe1c/logo.png")
            val = memory.getData("FaceDetected", 0)
            # if face is not detected
            if not val:
                got_face = False # if false
                prev_name = ""
            elif not got_face: # if false -> true, it true -> false, the previous case will trigger first
                got_face = True
                # read and print its shape info and ID
                # first field = TimeStamp
                timeStamp = val[0]
                # print "TimeStamp is: " + str(timeStamp)
                faceInfoArray = val[1]
                for j in range(len(faceInfoArray) - 1):
                    faceInfo = faceInfoArray[j]
                    # first field = shape info
                    faceShapeInfo = faceInfo[0]
                    # second field = extra info
                    faceExtraInfo = faceInfo[1]
                    # print "Face Info :  alpha %.3f - beta %.3f" % (faceShapeInfo[1], faceShapeInfo[2])
                    # print "Face Info :  width %.3f - height %.3f" % (faceShapeInfo[3], faceShapeInfo[4])
                    # print "Face Extra Info :" + str(faceExtraInfo)

                    # if face is recognised (faceExtraInfo[2] has 'Name')
                    if faceExtraInfo[2]:
                        name = str(faceExtraInfo[2])
                        if name and name != prev_name:
                            tts.say("Nice to see you " + name)
                        prev_name = name
                    # else face not recognised, learn face
                    else:
                        userID = userSession.getFocusedUser()
                        if userInfo.has(userID, 'name'):
                            name = str(userInfo.get(userID, 'name'))
                            # if name recognised, relearn face
                            if name in faceDetect.getLearnedFacesList():
                                if name and name != prev_name:
                                    faceDetect.reLearnFace(name)
                                    tts.say("Nice to see you again " + name)
                                prev_name = name
                            # else learn face
                            else:
                                if name and name != prev_name:
                                    faceDetect.learnFace(name)
                                    tts.say("Welcome to I R L Crossing " + name)
                                prev_name = name
                        # if no name
                        else:
                            prev_name = ""
                            tts.say("Hello, Welcome to I R L Crossing")

    except KeyboardInterrupt:
        # stop tracking
        tracker.stopTracker()
        tracker.unregisterAllTargets()
        # stopping the dialog engine
        dialog.unsubscribe('my_dialog_example')
        # Deactivating all topics
        dialog.deactivateTopic(topic_name)
        # now that the dialog engine is stopped and there are no more activated topics,
        # we can unload all topics and free the associated memory
        dialog.unloadTopic(topic_name)


if __name__ == '__main__':
    ip = "127.0.0.1"
    port = 9559
    # 127.0.0.1
    session = qi.Session()
    session.connect("tcp://" + ip + ":" + str(port))
    main(session)
