# Kimia Farma - Final Task PBI - Wilianto-Kurniawan

#membuat CTE untuk melakukan kategorisasi persentase gross laba berdasarkan kelompok harga
WITH gross_laba AS(
  SELECT 
    *,
    CASE 
      WHEN prd.price <= 50000 THEN 0.1 #persentase gross laba 10% untuk harga produk <=Rp50k
      WHEN prd.price >50000 AND prd.price <=100000 THEN 0.15 #persentase gross laba 15% untuk harga produk Rp50k-Rp100k
      WHEN prd.price >100000 AND prd.price <=300000 THEN 0.2 #persentase gross laba 20% untuk harga produk Rp100k-Rp300k
      WHEN prd.price >300000 AND prd.price <=500000 THEN 0.25 #persentase gross laba 25% untuk harga produk Rp300k-Rp500k
      ELSE 0.3 #persentase gross laba 300% untuk harga produk >Rp500k
    END AS persentase_gross_laba
  FROM `rakamin-kf-analytics-416103.kimia_farma.kf_product` prd
)

#membuat tabel analisis berisikan kolom-kolom penting
#dengan cara menggabungkan tabel final transaction, tabel kantor cabang dan tabel product
SELECT 
  trx.transaction_id, trx.date, trx.customer_name, trx.branch_id,
  cbg.branch_name, cbg.kota, cbg.provinsi,cbg.rating AS rating_cabang,
  prd.product_id, prd.product_name, prd.price AS actual_price,
  trx.discount_percentage,
  gl.persentase_gross_laba,
  (1-trx.discount_percentage)*prd.price AS net_sales, #perhitungan untuk net sales = (1-disc)*harga
  (gl.persentase_gross_laba-trx.discount_percentage)*prd.price AS net_profit,#perhitungan untuk net profit = (gross laba - disc)*harga
  trx.rating AS rating_transaksi
FROM `rakamin-kf-analytics-416103.kimia_farma.kf_final_transaction`AS trx #simpan nama tabel transaction sebagai trx
#simpan nama tabel kantor cabang sebagai cbg dan digabungkan dengan tabel trx
JOIN `rakamin-kf-analytics-416103.kimia_farma.kf_kantor_cabang`AS cbg ON trx.branch_id = cbg.branch_id 
#simpan nama tabel product sebagai prd dan digabungkan dengan tabel trx
JOIN `rakamin-kf-analytics-416103.kimia_farma.kf_product`AS prd ON trx.product_id = prd.product_id
#simpan tabel gross laba dari CTE dengan nama gl dan digabungkan dengan tabel trx
JOIN gross_laba AS gl ON trx.product_id = gl.product_id
