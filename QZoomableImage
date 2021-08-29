import QtQuick 2.15

Flickable
{
    id:flickable
    property int imageWidth
    property int imageHeight
    property alias source: mImage.source
    contentWidth: imageWidth
    contentHeight:imageHeight

    boundsBehavior: Flickable.StopAtBounds
    boundsMovement: Flickable.StopAtBounds


    states: [
        State {
            name: "state_StickToCenter" // state is used when content size is less than flickable size then content
            // center should stick to flickable center
            when : ( flickable.contentWidth < flickable.width || flickable.contentHeight< flickable.height )
            PropertyChanges {
                target: flickable.contentItem
                anchors.horizontalCenter:  width < flickable.width ? flickable.horizontalCenter : undefined
                anchors.verticalCenter:   height < flickable.height ? flickable.verticalCenter : undefined
            }
        }
    ]
    onStateChanged: { cancelFlick(); returnToBounds(); }


    Image
    {
        id:mImage
        width:flickable.contentWidth;
        height: flickable.contentHeight;
        source: flickable.imageSource
        fillMode: Image.PreserveAspectFit;
        Component.onCompleted:
        {
            imageWidth = mImage.paintedWidth;
            imageHeight = mImage.paintedHeight
        }

        autoTransform: true

        PinchArea
        {
            id:pinchArea
            anchors.fill: parent
            property bool zoomTriggeredFromPinchArea:false
            property point pinchCenter;


            onPinchStarted: zoomTriggeredFromPinchArea=true;
            onPinchUpdated:
            {
                var newZoomFactor = privateProperties.currentZoomFactor+ privateProperties.currentZoomFactor*(pinch.scale-1);
                pinchCenter =pinch.center;
                privateProperties.zoom(privateProperties.getBoundedScaleFactor(newZoomFactor))
            }

            onPinchFinished:
            {
                privateProperties.currentZoomFactor +=  privateProperties.currentZoomFactor*(pinch.scale-1);
                privateProperties.currentZoomFactor = privateProperties.getBoundedScaleFactor(privateProperties.currentZoomFactor);
                zoomTriggeredFromPinchArea=false;
            }

            MouseArea
            {
                id:mouseArea
                anchors.fill: parent
                propagateComposedEvents :true
                scrollGestureEnabled: false
                hoverEnabled: true

                onDoubleClicked:
                {
                    if(privateProperties.currentZoomFactor>1)resetScale();
                    else
                    {
                        var widthScale = (flickable.width+20)/mImage.width;
                        var heightScale = (flickable.height+20)/mImage.height;
                        var maxScale = Math.max(widthScale,heightScale);
                        if(maxScale>1)
                        {
                            privateProperties.pointOfDoubleClick = Qt.point(mouseX,mouseY);
                            privateProperties.useDoubleClickPoint = true;
                            privateProperties.currentZoomFactor = maxScale;
                        }
                    }
                }

                onWheel:
                {
                    if(wheel.modifiers===Qt.ControlModifier)
                    {
                        wheel.accepted=true;
                        var newZoomFactor;
                        if(wheel.angleDelta.y>0)
                            newZoomFactor = privateProperties.currentZoomFactor + (privateProperties.currentZoomFactor*privateProperties.zoomStepFactor);
                        else  newZoomFactor = privateProperties.currentZoomFactor - (privateProperties.currentZoomFactor*privateProperties.zoomStepFactor);
                        privateProperties.currentZoomFactor = privateProperties.getBoundedScaleFactor(newZoomFactor);
                        return;
                    }
                    wheel.accepted=false;
                }
            }
        }
    }

    QtObject
    {
        id : privateProperties
        property bool useDoubleClickPoint:false;
        property point pointOfDoubleClick;
        property real maxZoomFactor : 30.0
        property real zoomStepFactor :0.3;
        property real currentZoomFactor: 1
        property real minZoomFactor :1;
        property point scaleCenter : pinchArea.zoomTriggeredFromPinchArea
                                     ? pinchArea.pinchCenter : Qt.point(mouseArea.mouseX,mouseArea.mouseY);
        Behavior on currentZoomFactor {
            NumberAnimation { id:scaleNumberAnimation
                duration: pinchArea.zoomTriggeredFromPinchArea ? 0 : privateProperties.useDoubleClickPoint ?
                                                 Math.min(200*privateProperties.currentZoomFactor,500) : 200 ;
                onRunningChanged:  if(!running) privateProperties.useDoubleClickPoint=false;
            }
        }

        onCurrentZoomFactorChanged:
        {
            if(!pinchArea.zoomTriggeredFromPinchArea)
                zoom(currentZoomFactor);
        }

        function zoom(scaleFactor)
        {
            var targetWidth =  imageWidth*scaleFactor;
            var targetHeight  = imageHeight*scaleFactor;
            if(useDoubleClickPoint) resizeContent(targetWidth,targetHeight,mapToItem(mImage,pointOfDoubleClick));
            else resizeContent(targetWidth,targetHeight,scaleCenter);
            returnToBounds();
        }

        function getBoundedScaleFactor(ScaleFactor)
        {
            if(ScaleFactor>maxZoomFactor)ScaleFactor = maxZoomFactor;
            else if(ScaleFactor<minZoomFactor)ScaleFactor = minZoomFactor;
            return ScaleFactor;
        }
    }

    function resetScale()
    {
        privateProperties.pointOfDoubleClick = Qt.point(0,0);
        privateProperties.useDoubleClickPoint = true;
        privateProperties.currentZoomFactor = 1;
    }

}
