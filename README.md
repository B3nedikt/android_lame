# android_lame
A Small wrapper around the famous lame libary (http://lame.sourceforge.net/) for android based mostly on [this](http://developer.samsung.com/technical-doc/view.do;jsessionid=hlCGVWCRy8pwjsY5f4P8KLzPmr6fCZNvQhxGndXwRLTywbJT7vWX!404960129?v=T000000090) tutorial with some small changes to support newer android versions. 
#Installation
## Android Studio
Copy the .so file from releases to your source->main->jniLibs, if jniLibs does not exist create it.
## Build
Simply copy everything from this repository to source->main->jni and run the ndk-build script, as described in the tutorial above.
# Usage
Some definitions:
```java
    static {
        System.loadLibrary("mp3lame");
    }
    
    private native int encodeFile(String sourcePath, String targetPath);

    public static final int NUM_CHANNELS = 1;
    public static final int SAMPLE_RATE = 16000;
    public static final int BITRATE = 128;
    public static final int MODE = 1;
    public static final int QUALITY = 2;
    
    private AudioRecord mRecorder;
    private short[] mBuffer;
    
    private File mRawFile;
    private File mEncodedFile;
```
To initialize lame and start recording use:
```java
        public void start(){
        mRawFile = new File(...)
        mEncodedFile = new File(...)
        initRecorder();
        initEncoder(NUM_CHANNELS, SAMPLE_RATE, BITRATE, MODE, QUALITY);
        
        mRecorder.startRecording();
        
        startBufferedWrite(mRawFile);
        }
        
        private void initRecorder() {
        int bufferSize = AudioRecord.getMinBufferSize(SAMPLE_RATE, AudioFormat.CHANNEL_IN_MONO,
                AudioFormat.ENCODING_PCM_16BIT);
        mBuffer = new short[bufferSize];
        mRecorder = new AudioRecord(MediaRecorder.AudioSource.MIC, SAMPLE_RATE, AudioFormat.CHANNEL_IN_MONO,
                AudioFormat.ENCODING_PCM_16BIT, bufferSize);
    }
        
```
Finally encode the file and release everything:
```java
        public void release(){
        File mEncodedFile = new File(mFileName);
        mRawFile = mCacheFile.getMergedFile();
        int result = encodeFile(mRawFile.getAbsolutePath(), mEncodedFile.getAbsolutePath());
        if (result == 0) {
            Log.d("NativeRecorder", "Encoded to " + mEncodedFile.getName());
        }
            
        mRecorder.release();
        destroyEncoder();
        }
        
        private void startBufferedWrite(final File file) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                DataOutputStream output = null;
                try {
                    output = new DataOutputStream(new BufferedOutputStream(new FileOutputStream(file)));
                    while (mIsRecording) {
                        double sum = 0;
                        int readSize = mRecorder.read(mBuffer, 0, mBuffer.length);
                        for (int i = 0; i < readSize; i++) {
                            output.writeShort(mBuffer[i]);
                            sum += mBuffer[i] * mBuffer[i];
                        }

                        if (readSize > 0) {
                            mAmplitude = (int) (sum / readSize);
                        }
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                } finally {
                    if (output != null) {
                        try {
                            output.flush();
                        } catch (IOException e) {
                            e.printStackTrace();
                        } finally {
                            try {
                                output.close();
                            } catch (IOException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }
            }
        }).start();
    }
    
```
#License
Lame is licensed under the LGPL: (http://lame.sourceforge.net/license.txt)
