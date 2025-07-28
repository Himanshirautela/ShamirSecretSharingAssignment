# ShamirSecretSharingAssignment

import java.io.*;
import java.math.BigDecimal;
import java.math.BigInteger;
import java.util.*;
import org.json.JSONObject;
public class ShamirSecretFinder {
    static class Point {
        BigDecimal x;
        BigDecimal y;
        Point(BigDecimal x, BigDecimal y) {
            this.x = x;
            this.y = y;
        }
    }
    public static void main(String[] args) throws Exception {
        String[] files = { "testcase1.json", "testcase2.json" };
        for (String file : files) {
            JSONObject obj = new JSONObject(new String(java.nio.file.Files.readAllBytes(new File(file).toPath())));
            int k = obj.getJSONObject("keys").getInt("k");

            List<Point> points = new ArrayList<>();
            for (String key : obj.keySet()) {
                if (key.equals("keys")) continue;
                int x = Integer.parseInt(key);
                JSONObject pair = obj.getJSONObject(key);
                int base = Integer.parseInt(pair.getString("base"));
                BigDecimal y = new BigDecimal(new BigInteger(pair.getString("value"), base));
                points.add(new Point(BigDecimal.valueOf(x), y));
            }
            BigDecimal secret = lagrangeInterpolation(points.subList(0, k));
            System.out.println("Secret (c) for " + file + " = " + secret.toBigInteger().toString());
        }
    }
    static BigDecimal lagrangeInterpolation(List<Point> points) {
        BigDecimal result = BigDecimal.ZERO;

        for (int j = 0; j < points.size(); j++) {
            BigDecimal xj = points.get(j).x;
            BigDecimal yj = points.get(j).y;

            BigDecimal term = yj;
            for (int i = 0; i < points.size(); i++) {
                if (i == j) continue;
                BigDecimal xi = points.get(i).x;
                BigDecimal numerator = xi.negate();
                BigDecimal denominator = xj.subtract(xi);
                term = term.multiply(numerator).divide(denominator, 20, BigDecimal.ROUND_HALF_UP);
            }
            result = result.add(term);
        }
        return result.setScale(0, BigDecimal.ROUND_HALF_UP);
    }
}
