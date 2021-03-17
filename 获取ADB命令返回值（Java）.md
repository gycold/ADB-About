```java

public static boolean ExuseCmd(String cmdStr){
        Process process = null;
        DataOutputStream os = null;
        try {
            process = Runtime.getRuntime().exec("su");
            os = new DataOutputStream(process.getOutputStream());
            os.writeBytes(cmdStr + "\n");
            os.writeBytes("exit\n");
            os.flush();
		//获取返回值
        InputStream is = process.getInputStream();
        BufferedReader reader = new BufferedReader(new InputStreamReader(is));
     	StringBuffer strb = new StringBuffer();
        String line;
        while ((line = reader.readLine()) != null) {
            strb.append(line + "\n");
            Log.d("cmd",""+strb);
        }
        is.close();
        reader.close();
        process.waitFor();
        process.destroy();
    } catch (Exception e) {
        return false;
    } finally {
        try {
            if (os != null) {
                os.close();
            }
            if (process != null) {
                process.destroy();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
     return true;
}
```
